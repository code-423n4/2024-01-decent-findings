
When a direct swap-and-execute operation is requested the function `swapAndExecute` is called. The function accepts either native or ERC20 tokens.

Transactions through The Box typically provides six possible forms, the direct execution allows to use a native token as `tokenIn` and `tokenOut`.

When a native token is specified as `tokenIn` the Box will execute the  codepath in following function `performSwap`:

```solidity
if (swapParams.tokenIn == address(0)) {
	require(msg.value >= swapParams.amountIn, "not enough native");
	wrapped.deposit{value: swapParams.amountIn}();
	[...]
}
(tokenOut, amountOut) = swapper.swap(swapInstructions.swapPayload);
if (tokenOut == address(0)) {
	wrapped.withdraw(amountOut);
}
```

In case the `tokenOut` is a native token too, its value is `address(0)`, then the protocol executed unnecessary transfer operations that cost gas.

When `tokenIn == tokenOut == address(0)` (specifying native token as `tokenIn` and `tokenOut`) the protocol has the potential to save on gas expenses. This can be achieved by avoiding the token transfers to the Swapper and back to the UTB. Instead, the protocol can directly call the function `execute`.