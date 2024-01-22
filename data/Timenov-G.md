## Summary

[G-01] Unnecessary `string.concat`.

### [G-01] Unnecessary `string.concat`.
There are 2 places, where `string.concat` function is used in the `require` error message. There is no need for that as there is only one string used. There is a possibility the development team forgot to add the second string, but I am reporting this as a GAS finding, since the solution I will give will save gas.

```solidity
        require(
            dstAddr != address(0),
            string.concat("dst chain address not set ") @audit remove string.concat
        );
```

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/StargateBridgeAdapter.sol#L157C1-L160C11

```solidity
        require(
            destinationBridgeAdapter[dstChainId] != address(0),
            string.concat("dst chain address not set ") @audit remove string.concat
        );
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L91-L94