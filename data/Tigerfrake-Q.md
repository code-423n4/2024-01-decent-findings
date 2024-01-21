# [01] Floating pragma

The disadvantage of using a floating pragma is that it can lead to inconsistencies and potential bugs. When a contract is compiled with a different compiler version, it might produce different bytecode due to changes in the compiler, even if the source code looks the same. This can lead to unexpected behavior if the contract is deployed on a network with a different compiler version than the one it was tested with.

```Solidity
pragma solidity ^0.8.0;
```
Moreover, if the contract uses features that were introduced in a newer compiler version, and it gets compiled with an older compiler version, it could fail or behave differently than expected.

### Instances:
All contracts 

### Recommendation:
It's generally recommended to lock the compiler version to avoid these issues.
```Solidity
pragma solidity 0.8.0;
```

# [02] Missing Zero- address Validation

### Description
Lack of `zero-address validation` on `address parameters` may lead to `transaction reverts, waste gas, require resubmission of transactions and may even force contract redeployments` in certain cases within the protocol.

### Instances:
- https://github.com/code-423n4/2024-01-decent/blob/main/src%2Fswappers%2FUniSwapper.sol#L104
- https://github.com/code-423n4/2024-01-decent/blob/main/src%2Fswappers%2FUniSwapper.sol#L125

### Recommendation
Consider adding explicit zero-address validation on input parameters of address type.

# [03] Missing or Incomplete NatSpec

### Description
Some functions are missing `@notice/@dev` NatSpec comments for the function, `@param` for all/some of their parameters and `@return` for return values. Given that NatSpec is an important part of code documentation, this affects code comprehension, auditability and usability.

### Instances:
All contracts

### Recommendation
> Consider adding in full NatSpec comments for all functions to have complete code documentation for future use.

# [04] Undocumented Value Literal
### Description:
The value literal 100000 is hardcoded to a constant variable GAS_FOR_RELAY:

```Solidity
GAS_FOR_RELAY = 100000;
```
However, it is undocumented and unclearly depicted.

### Instance:
https://github.com/decentxyz/decent-bridge/blob/main/src%2FDecentEthRouter.sol#L96

### Recommendation:
We advise the special underscore (_) separator to be applied to it (i.e. 100000 would become 100_000) 
