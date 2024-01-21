# Summary

## Low-Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| L&#x2011;01 | Users can swap and execute without paying protocol fees | 1 |
| L&#x2011;02 | Signature malleability of EVM's `ecrecover` in collectFees() | 1 |
| L&#x2011;03 | Signature replay due to no protocol controlled variables in hash | 1 |

## [L&#x2011;01] Users can swap and execute without paying protocol fees
### Lines of code
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L108
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L311
### Description
As there is no access control modifier on `UTB.sol::receiveFromBridge`, it can be called by anyone. Both of these functions simply call `_swapAndExecute`, making them interchangeable in the case of ERC20 swaps. However, it's important to note that `UTB.sol::receiveFromBridge` does not have a `payable` modifier, so it cannot be used for Ether-based calls. Additionally, since `UTB.sol::receiveFromBridge` does not utilize the `retrieveAndCollectFees` modifier, fees are not collected in this function.

### Mitigation steps
Consider adding a modifier to limit access `UTB.sol::receiveFromBridge` to approved bridge adapters only.

## [L&#x2011;02] Signature malleability of EVM's `ecrecover` in collectFees()
### Lines of code
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L53

### Description
EVM's `ecrecover` is susceptible to a signature malleability which allows replay attacks. In case of fee increase, users may reuse previous fees by supplying another signature, valid for the supplied `constructedHash`.

Please see this document for the reference: https://swcregistry.io/docs/SWC-117

### Mitigation steps
Consider using `recover` function from OpenZeppelin ECDSA library: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.9/contracts/utils/cryptography/ECDSA.sol


## [L&#x2011;03] Signature replay due to no protocol controlled variables in hash
### Lines of code
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L115C38-L115C68
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L49-L50

### Description
In the event of a fee increase, users may continue to use the old fees as long as they submit the same data in the `SwapAndExecuteInstructions` or `BridgeInstructions` structures. The reason for this is the absence of parameters in a signed hash that are controlled by the protocol or, specifically, the signer.

### Mitigation steps
Consider adding  `deadline` (i.e. timestamp) into FeeStructure struct and check it in the `UTBFeeCollector.sol::collectFees` function.

