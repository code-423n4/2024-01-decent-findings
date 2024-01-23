# Summary

| Risk   | Title                                                                       |
|--------|-----------------------------------------------------------------------------|
| [L-01] | Refund calls can fail and funds will be locked inside `UTBExecutor`          |
| [L-02] | Anyone can swap freely without paying fees when `signer` is not set         |
| [L-03] | Deadline not set when swapping                                              |
| [L-04] | Excess value is not refunded to the user when swapping native               |
| [L-05] | `onlyIfWeHaveEnoughReserves` fails if there are enough reserves             |
| [L-06] | Funds can be locked when a user adds `weth` liquidity with some `msg.value` |
| [L-07] | Approvals are not canceled after a partial transfer                        |


## Low Risk

### [L-01] Refund calls can fail and funds will be locked inside `UTBExecutor`

When a `target` call fails, a refund should be issued to the user's `refund` address.

However, it's not _guaranteed_ that the refund call will succeed: in this case, these funds will be permanently locked inside `UTBExecutor`:

```solidity
	if (token == address(0)) {
	    (success, ) = target.call{value: amount}(payload);
	    if (!success) {
@>	        (refund.call{value: amount}(""));
	    }
	    return;
	}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L51-L55


### [L-02] Anyone can swap freely without paying fees when `signer` is not set

Fees are collected each time a user performs a swap, which are collected through `collectFees`:

```solidity
	function collectFees(
	    FeeStructure calldata fees,
	    bytes memory packedInfo,
	    bytes memory signature
	) public payable onlyUtb {
	    bytes32 constructedHash = keccak256(
	        abi.encodePacked(BANNER, keccak256(packedInfo))
	    );
	    (bytes32 r, bytes32 s, uint8 v) = splitSignature(signature);
@>	    address recovered = ecrecover(constructedHash, v, r, s);
@>	    require(recovered == signer, "Wrong signature"); //@audit L - no check for recovered == 0?
	    if (fees.feeToken != address(0)) {
	        IERC20(fees.feeToken).transferFrom(
	            utb,
	            address(this),
	            fees.feeAmount
	        );
	    }
	}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L53-L54

`signer` is initially set to the zero address, so the expected behavior seems to be that every transaction should revert until the `signer` address is set to a valid address.

However, users can pass a malformed `signature` so that `ecrecover` returns zero: in this case the `require` will pass, and users will not pay any fees.

### [L-03] Deadline not set when swapping

The deadline is not set when swapping, as it's commented out on `ExactOutputParams`, and it's missing on `ExactInputParams`.

```solidity
	IV3SwapRouter.ExactOutputParams memory params = IV3SwapRouter
	    .ExactOutputParams({
	        path: swapParams.path,
	        recipient: address(this), //@audit L - deadline not set
	        //deadline: block.timestamp,
	        amountOut: swapParams.amountOut,
	        amountInMaximum: swapParams.amountIn
	    });
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L153


```solidity
	IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
	    .ExactInputParams({
	        path: swapParams.path,
	        recipient: address(this),
	        amountIn: swapParams.amountIn,
	        amountOutMinimum: swapParams.amountOut
	    });
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L129-L135

This will cause unfavorable swaps to the users, especially if the TX stays on the mempool for a long time.

It's recommended to add a deadline parameter, preferably as a `swapParams` field, so the user can set it.

### [L-04] Excess value is not refunded to the user when swapping native

When the `tokenIn` is native, the `msg.value` needs to be greater or equal than `swapParams.amountIn`:

```solidity
	if (swapParams.tokenIn == address(0)) {
@>	    require(msg.value >= swapParams.amountIn, "not enough native"); //@audit L - msg.value excess not refunded
	    wrapped.deposit{value: swapParams.amountIn}();
	    swapParams.tokenIn = address(wrapped);
	    swapInstructions.swapPayload = swapper.updateSwapParams(
	        swapParams,
	        swapInstructions.swapPayload
	    );
	}
```

However, when `msg.value` is greater, the excess is not refunded to the user.

Consider either changing the `require` so that the `msg.value` is strictly equal to the `amountIn`, or as alternative, refund the excess amount.

### [L-05] `onlyIfWeHaveEnoughReserves` fails if there are enough reserves

Due to an off-by-one error, this modifier will fail even when the balance is sufficient.

The balance needs to be `amount` + `1 wei` to avoid a revert. This can cause some issues when the balance is zero, and a precise `amount` is sent to a contract, if the caller doesn't remember to top it with 1 additional `wei`:

```solidity
modifier onlyIfWeHaveEnoughReserves(uint256 amount) {
    require(weth.balanceOf(address(this)) > amount, "not enough reserves"); //@audit L - fails with enough reserves
    _;
}
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L50-L53

Consider modifying the `require` so that it doesn't fail when `weth.balanceOf(address(this)) >= amount` instead.

### [L-06] Funds can be locked when a user adds `weth` liquidity with some `msg.value`

A user can call by mistake `addLiquidityWeth` with some `msg.value`: these funds will not be used and they will be locked inside the contract

```solidity
function addLiquidityWeth(
        uint256 amount
) public payable userDepositing(amount) {
    weth.transferFrom(msg.sender, address(this), amount);
    dcntEth.mint(address(this), amount);
}
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L302-L310

Consider removing `payable` from the function signature.


### [L-07] Approvals are not canceled after a partial transfer

It's not guaranteed that the target will consume the entire allowance after a call:

```solidity
	uint256 balanceBefore = weth.balanceOf(address(this));
	weth.approve(target, amount); 

    (bool success, ) = target.call(callPayload); 
    //@audit L - approval not canceled after the call
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L30-L33

Consider adding a `weth.approve(target, 0);` after the actual call to avoid any leftover approvals.