## 1. There is an approval of tokens that is given by `Uniswapper.sol` to `uniswap_router` in `UniSwapper.sol/swapExactOut()`.
[Link to the function](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L143)

This is due to the fact that the approval is given for `amountInMaximum` number of tokens but the actual amount taken out is less than that and this will leave behind a small approval to `uniswap_router`. This violates the guidelines that are written in the docs 
```
The contracts are not intended to hold on to any funds or unnecessary approvals
```
This will be better to include this line at the end of the function implementation
```
IERC20(swapParams.tokenIn).approve(uniswap_router,0);
```

## 2. The collectFees() function in UTBFeeCollector.sol can be executed prior to setting the signer and will not impose any fees.
The setSigner() function in UTBFeeCollector.sol is susceptible to front-running. This allows users to bypass the collectFees() function by supplying arbitrary values for v, r, s. Consequently, the recovered address becomes address(0), matching the initial/default signer address(just after deployment), enabling the bypass of the collectFees() function without incurring any fees.