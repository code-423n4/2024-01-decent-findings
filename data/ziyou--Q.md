| |Issue|Instances| Gas Savings
|-|:-|:-:|:-:|
| [[G-01](#g-01)] | The modifier 'userDepositing' lacks a check for the 'amount' parameter being zero, resulting in wasted gas. | 1| 0|

### [G-01]The modifier 'userDepositing' lacks a check for the 'amount' parameter being zero, resulting in wasted gas.





```solidity

File: lib/decent-bridge/src/DecentEthRouter.sol

55:     modifier userDepositing(uint256 amount) {
        balanceOf[msg.sender] += amount;
        _;
    }


```




*GitHub* : https://github.com/code-423n4/2024-01-decent/blob/main//lib/decent-bridge/src/DecentEthRouter.sol#L55-L58

