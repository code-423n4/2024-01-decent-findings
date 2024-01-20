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

### Recommendation
Consider adding explicit zero-address validation on input parameters of address type.

