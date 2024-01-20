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
