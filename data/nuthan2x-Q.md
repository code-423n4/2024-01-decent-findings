 |no. |Issue
 |-|:-
| [[L-01](#l-01)] | Swap fees are not collected on `UniSwapper::swapExactIn`|
| [[L-02](#l-02)] | Swap transactions fail if `reveiver == UTB` is not enforced|
| [[L-03](#l-03)] | Refund calls are not validated if transfer is succeeded|
| [[L-04](#l-04)] | A user can use any feeToken of choice to pay as its permissionless|


### [L-01]<a name="l-01"></a> swap fees are not collected on [UniSwapper::swapExactIn](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/swappers/UniSwapper.sol#L123) call

### Impact
- swap fees are collected on bridge / swap actions. They are collected when users brisge/swap through the router.
- But, users can also swap directly interacting with swapper adapter contract and swap away, without paying the fees to decent.

### Recommendation

- Modify [UniSwapper::swapExactIn](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/swappers/UniSwapper.sol#L123) and [UniSwapper::swapExactOut](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/swappers/UniSwapper.sol#L143) as rivate instead of public  as shown below.

```diff  
function swapExactIn(
        SwapParams memory swapParams, // SwapParams is a struct
        address receiver
-   ) public payable routerIsSet returns (uint256 amountOut) {
+   ) private payable routerIsSet returns (uint256 amountOut) {
        swapParams = _receiveAndWrapIfNeeded(swapParams);

        IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
            .ExactInputParams({
                path: swapParams.path,
                recipient: address(this),
                amountIn: swapParams.amountIn,
                amountOutMinimum: swapParams.amountOut
            });

        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
        amountOut = IV3SwapRouter(uniswap_router).exactInput(params);

        _sendToRecipient(receiver, swapParams.tokenOut, amountOut);
    }
```

### [L-02]<a name="l-02"></a> swap transactions fail if `reveiver == UTB` is not enforced

### Impact
- The swapped tokens come from LP pool into [UniSwapper](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/swappers/UniSwapper.sol#L13) contract and transferred to receiver.
- But these swap transactions are made by [UTB::swapAndExecute](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L108) and they need to continye executing the payoads after the swap action.
- And these payload executions fail due to the lack of tokens, because the tokens are transaferred to receiver (some EOA) instead of UTB contract.

### Recommendation

- Modify [UniSwapper::swap](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/swappers/UniSwapper.sol#L58-L77) as below.
```diff  
    function swap(
        bytes memory swapPayload
    )
        external
        onlyUtb
        returns (address tokenOut, uint256 amountOut)
    {
        (SwapParams memory swapParams, address receiver, address refund) = abi
            .decode(swapPayload, (SwapParams, address, address));

+       require(receiver == utb, "invalid receiver");

        tokenOut = swapParams.tokenOut;
        if (swapParams.path.length == 0) {
            return swapNoPath(swapParams, receiver, refund);
        }
        if (swapParams.direction == SwapDirection.EXACT_IN) {
            amountOut = swapExactIn(swapParams, receiver);
        } else {
            swapExactOut(swapParams, receiver, refund);
            amountOut = swapParams.amountOut;
        }
    }
```

### [L-03]<a name="l-03"></a> refund calls are not validated if transfer is succeeded

### Impact
- [UTBExecutor::execute] executes a payload transaction after swapping, and if any external call fails, then the funds are refunded to `refund address`
- But those call's transaction success is not validated. The refund address might not accept ether.

### Recommendation

- So check if it reverts by bool validation and push WETH tokens
- Modify [UTBExecutor::execute](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/swappers/UniSwapper.sol#L58-L77) as below.
```diff  
        function execute(
        address target,
        address paymentOperator,
        bytes memory payload,
        address token,
        uint amount,
        address payable refund,
        uint extraNative
    ) public onlyOwner {
        bool success;
        if (token == address(0)) {
            (success, ) = target.call{value: amount}(payload);
            if (!success) {
-               (refund.call{value: amount}(""));
+               (bool s, ) = (refund.call{value: amount}(""));
+               if(!s) {
+                    WETH.deposit{value : amount}();
+                    weth.transfer(refund, amount);
+               }
            }
            return;
        }

        uint initBalance = IERC20(token).balanceOf(address(this));

        IERC20(token).transferFrom(msg.sender, address(this), amount);
        IERC20(token).approve(paymentOperator, amount);

        if (extraNative > 0) {
            (success, ) = target.call{value: extraNative}(payload);
            if (!success) {
-               (refund.call{value: extraNative}(""));
+               (bool s, ) = (refund.call{value: extraNative}(""));
+               if(!s) {
+                    WETH.deposit{value : extraNative}();
+                    weth.transfer(refund, extraNative);
+               }
        } else {
            (success, ) = target.call(payload);
        }

        uint remainingBalance = IERC20(token).balanceOf(address(this)) -
            initBalance;

        if (remainingBalance == 0) {
            return;
        }

        IERC20(token).transfer(refund, remainingBalance);
    }
```









### [L-04]<a name="l-04"></a> A user can use any feeToken of choice to pay as its permissionless

### Impact
- A userc an set an EOA as feetoken and whatever the approve, transfer, transferFrom call will return true and fee will not be collected at end of swap call.
- If interface ienforced, the the user will deploy a new custom token and then he can swap aby avoiding paying fees.

### Recommendation

- Enforce a whitelist of tokens to be allowed either offchain/on contract level itself
