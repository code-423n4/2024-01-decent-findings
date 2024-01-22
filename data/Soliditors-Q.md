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

`execute` function in UTBExecutor contract is meant to be called by the UTB contract. Indeed, we can see in Tasks contract in scripts.sol that the ownership of UTBExecutor is transferred to UTB contract.

```
utbExecutor = new UTBExecutor();
utbExecutor.transferOwnership(utb);
```

We can see that the first `execute` function is called, which is as follows:
```
    function execute(
        address target,
        address paymentOperator,
        bytes memory payload,
        address token,
        uint256 amount,
        address payable refund
    ) public payable onlyOwner {
        return execute(target, paymentOperator, payload, token, amount, refund, 0);
    }
```
This function will call the other `execute`, setting to a 0 hardcoded value the `extraNative` parameter. 

This means that `extraNative` will never have a non zero value. Hence, the following code will never be reached : 

```
    if (extraNative > 0) {
            (success,) = target.call{value: extraNative}(payload);
            if (!success) {
                (refund.call{value: extraNative}(""));
            }
        }
```

If this is indeed the expected behaviour, the code should be simplify, leaving only one `execute` function, for example : 

```
 function execute(
        address target,
        address paymentOperator,
        bytes memory payload,
        address token,
        uint256 amount,
        address payable refund,
    ) public onlyOwner {
        bool success;
        if (token == address(0)) {
            (success,) = target.call{value: amount}(payload);
            if (!success) {
                (refund.call{value: amount}(""));
            }
            return;
        }

        uint256 initBalance = IERC20(token).balanceOf(address(this));

        IERC20(token).transferFrom(msg.sender, address(this), amount);
        IERC20(token).approve(paymentOperator, amount);
 
        (success,) = target.call(payload);

        uint256 remainingBalance = IERC20(token).balanceOf(address(this)) - initBalance;

        if (remainingBalance == 0) {
            return;
        }

        IERC20(token).transfer(refund, remainingBalance);
    } 
```

## [NC‑01]


## [L‑01]