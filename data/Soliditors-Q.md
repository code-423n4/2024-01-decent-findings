## [NC‑01] useless bool parameter in `performSwap` function in UTB contract

Two functions in UTB contract use the name `performSwap`, and both of them are private. `performSwap(SwapInstructions memory swapInstructions)` calls `performSwap(SwapInstructions memory swapInstructions, bool retrieveTokenIn)`, while setting systematically `retrieveTokenIn` to true.

There is no situation in which `retrieveTokenIn` will be set to false. Hence, ``performSwap(SwapInstructions memory swapInstructions, bool retrieveTokenIn)` should be removed, and the code could be simplified as follows: 

```
    function performSwap(SwapInstructions memory swapInstructions)
        private
        returns (address tokenOut, uint256 amountOut)
    {
        ISwapper swapper = ISwapper(swappers[swapInstructions.swapperId]);

        SwapParams memory swapParams = abi.decode(swapInstructions.swapPayload, (SwapParams));

        if (swapParams.tokenIn == address(0)) {
            require(msg.value >= swapParams.amountIn, "not enough native");
            wrapped.deposit{value: swapParams.amountIn}();
            swapParams.tokenIn = address(wrapped);
            swapInstructions.swapPayload = swapper.updateSwapParams(swapParams, swapInstructions.swapPayload);
        } else {
            IERC20(swapParams.tokenIn).transferFrom(msg.sender, address(this), swapParams.amountIn);
        }

        IERC20(swapParams.tokenIn).approve(address(swapper), swapParams.amountIn);

        (tokenOut, amountOut) = swapper.swap(swapInstructions.swapPayload);

        if (tokenOut == address(0)) {
            wrapped.withdraw(amountOut);
        }
    }
```

## [NC‑01]

## [L‑01]