# [LOW-1] UniSwapper.swapNoPath() does not work for different tokens

[Uniswapper.swapNoPath()](https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/swappers/UniSwapper.sol#L100-L121) will not work if the tokens provided are different. It expects that `amountIn >= amountOut`. Also if the decimals are different, it will send wrong amount of tokenOut: 
```solidity
 _sendToRecipient(receiver, swapParams.tokenOut, amt2Recipient);
```

# [LOW-2] UTBExecutor.execute() may lead to loss of funds if called on address without code

[`UTBExecutor.execute()`](https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L41-L81) can be called on address without code. This call will be successfull and a refund will not happen.

```solidity
 (success, ) = target.call{value: amount}(payload);
            if (!success) {
                (refund.call{value: amount}(""));
 }
```

# [LOW-3] `PUSH0` is not supported on some chains
Some chains, like Polygon, do not support the `PUSH0` opcode which is being used when contracts are compiled with 0.8.20 or above.

# [LOW-4] Hardcoded relay gas
[`DecentEthRouter._getCallParams()`](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L96) has the gas value for the relay hardcoded. This is bad because if the gas costs change in the future, the protocol may stop work correctly.

```solidity
        uint256 GAS_FOR_RELAY = 100000;
```
