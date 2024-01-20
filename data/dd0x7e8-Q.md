# QA Report

## Lack of `deadline` param in `UniSwapper`

## Lines 
https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/swappers/UniSwapper.sol#L129-L135

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/swappers/UniSwapper.sol#L149-L156

## Description
The `UniSwapper` lacks a `deadline` parameter, which may lead to potential fund losses. Since front-running is an integral risk in automated market maker (AMM) platforms, setting a deadline is essential to prevent the possibility of a transaction being deliberately withheld and executed at an unfavorable time. Without this temporal constraint, validators might have an incentive to delay processing the transaction until the slippage reaches its peak, thereby extracting maximum profit to the detriment of the user's transaction.

A validator might postpone the transaction until market conditions, such as slippage, become favorable or execute their own transaction first, effectively front-running the original user's transaction for potential gain.

```solidity=129
        IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
            .ExactInputParams({
                path: swapParams.path,
                recipient: address(this),
                amountIn: swapParams.amountIn,
                amountOutMinimum: swapParams.amountOut
            });
```

```solidity=149
        IV3SwapRouter.ExactOutputParams memory params = IV3SwapRouter
            .ExactOutputParams({
                path: swapParams.path,
                recipient: address(this),
                //deadline: block.timestamp,
                amountOut: swapParams.amountOut,
                amountInMaximum: swapParams.amountIn
            });
```

## Recommendation
It's recommended to add a proper `deadline` param according to [parameter-structs](https://docs.uniswap.org/contracts/v3/reference/periphery/interfaces/ISwapRouter#parameter-structs).