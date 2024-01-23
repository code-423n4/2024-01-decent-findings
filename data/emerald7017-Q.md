Approvals are granted at these main sites.

- [UTB](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol): When swapping tokens before bridge/swap 
- [UTBExecutor](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol): For token transfers
- [UniSwapper](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol): During swaps
- Bridge adapters: Before bridging tokens

**Approval Amounts**

The approval amounts match the intended transfer amounts at each call site. No signs of approving excess amounts.

For example:

```solidity
token.approve(spender, transferAmount); 
```

**Reset Logic**

**However, there is no active effort to reset allowances after transfers or account for drainage** 

For example by calling `approve(0)` or `allowance()` checks.

This accumulates allowance over time.

**Extreme Scenarios**

If tokens get self-destructed after approval, the spender contracts lose access to those tokens permanently.

No fundamental risks, _but suboptimal allowance hygiene_.

**Recommendations**

Actively resetting allowances to 0 after intended transfers can improve hygiene.