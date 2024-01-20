# [LOW-1] UniSwapper.swapNoPath() does not work for different tokens

[Uniswapper.swapNoPath()](https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/swappers/UniSwapper.sol#L100-L121) will not work if the tokens provided are different. It expects that `amountIn >= amountOut`. Also if the decimals are different, it will send wrong amount of tokenOut: 
```solidity
 _sendToRecipient(receiver, swapParams.tokenOut, amt2Recipient);
```