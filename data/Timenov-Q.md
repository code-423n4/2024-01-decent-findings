## Summary

[I-01] Unnecessary cast to the same type.
[L-01] Possible readonly re-entrancy.

### [I-01] Unnecessary cast to the same type.
There is no need to do explicit cast to same type. Here `bridgeExecutor` is of type address.

```solidity
    msg.sender == address(bridgeExecutor)
```

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/BaseAdapter.sol#L13

### [L-01] Possible readonly re-entrancy.
In `DecentEthRouter.sol`, the modifier `userIsWithdrawing` subtracts the amount at the end of the function. This could lead to readonly re-entrancy.

```solidity
    modifier userIsWithdrawing(uint256 amount) {
        uint256 balance = balanceOf[msg.sender];
        require(balance >= amount, "not enough balance");
        _;
        balanceOf[msg.sender] -= amount; @audit problem
    }
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L60-L65

This modifier is used in `removeLiquidityEth` and `removeLiquidityWeth`. This PoC will demonstrate how the attack will work:

1. Comment the lines that will not be needed to make it easier.

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L308-L309

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L316-L317

2. Create folder `audit` in the `test` folder.
3. Create file `Attack.sol` in the `audit` folder.
4. Copy from this [Gist](https://gist.github.com/Pavel2202/db3a8ebbaf57db662af3e975f3c5786d) and paste into `Attack.sol`.
5. Create test file `DecentEthRouterTest.t.sol` in `audit` folder.
6. Copy from this [Gist](https://gist.github.com/Pavel2202/8802b5c0a700858b1cf6d69ce75d1319) and paste into `DecentEthRouterTest.t.sol`.
7. Run `forge test --match-test test_readonly_reentrancy` in the terminal.
8. Test passes:

```solidity
[PASS] test_readonly_reentrancy() (gas: 56326)
```

To fix this issue, do the following changes to `userIsWithdrawing` modifier:

```diff
    modifier userIsWithdrawing(uint256 amount) {
        uint256 balance = balanceOf[msg.sender];
        require(balance >= amount, "not enough balance");
+       balanceOf[msg.sender] -= amount;
        _;
-       balanceOf[msg.sender] -= amount;
    }
```