## `retrieveTokenIn` will be `True` in all cases.

In the function [function performSwap(SwapInstructions memory swapInstructions,bool retrieveTokenIn)](https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTB.sol#L63-L88), the param input `retrieveTokenIn` will always be `true` in the contract scope.

```
    function performSwap(
        SwapInstructions memory swapInstructions,
        bool retrieveTokenIn
    ) private returns (address tokenOut, uint256 amountOut) {
        ...
    }
```

Consider removing this setting and modifying the code to make it more clear.