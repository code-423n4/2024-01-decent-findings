## [L-01] Signature verification check can be skipped in `retrieveAndCollectFees()`

The address of feeCollector is not set it the constructor, but in a separate method, and that method does not do input verification:
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L44

When a user attempts a swap in `swapAndExecute()`, the modifier `retrieveAndCollectFees()` is called:
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L115

It checks `if (address(feeCollector) != address(0))`.
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L228

If the address of `feeCollector` is not set the signature verification will not happen, so users can send any swap instructions they want. This can happen either if a user attempts a swap before the actual transaction call to `setFeeCollector()` is made or if the admins maliciosly/recklessly set it to address(0).

A user is required to approve an amount of tokens to be transferred to the UTB contract for a swap. The swap input has a `target` variable, which is the receiver of the `tokenOut` after the swap. This is verified by the signature, so if it is not checked the `target`(receiver) can be set by a malicios user to his own address in a transaction frontrunning a swap:
- https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L153 -> https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L52
This way an attacker can steal funds from a user.

Add `else { revert() }` to the if statement in `retrieveAndCollectFees()`. This will cost practically no gas and will completely prevent this from happening.
Alternatively add a call to `setFeeCollector()` in the constructor. Then require that the input in `setFeeCollector()` is not the zero address.

## [L-02] Call quoteLayerZero() to get a quote for the gas fee for the cross chain swap

The stargate documentation explains that you should use `quoteLayerZero()` to get the fee that should be sent with the swap: https://stargateprotocol.gitbook.io/stargate/developers/how-to-swap

Instead, the protocol currently sends all of the provided native token value:

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L170
```js
    function callBridge(...) private {
        router.swap{value: msg.value}(
            .
            .
            .
        );
    }
```
Use the value provided by the quote instead of relying on a refund by the external protocol and then refund the redundant amount to the user.