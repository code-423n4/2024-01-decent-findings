## Description:
In the UTB.sol contract, mappings are used to associate addresses with SWAPPER_ID and BRIDGE_ID, which are currently defined as uint8 data types. Given that these identifiers can only take values of 0 or 1, the mappings can be made more gas-efficient by using bool instead of uint8. This change aligns with the binary nature of the identifiers and may lead to reduced gas consumption.

## Impact:
The current implementation uses a larger data type (uint8) than necessary for SWAPPER_ID and BRIDGE_ID, resulting in potentially higher gas costs. While the impact may be marginal for individual transactions, gas savings can accumulate, especially in scenarios with high transaction volume.

## Mitigation Steps:
Consider modifying the mappings as follows:

```solidity
mapping(bool => address) public swappers;
mapping(bool => address) public bridgeAdapters;
```
This change ensures that the mappings are more gas-efficient, as bool can directly represent the binary nature of SWAPPER_ID and BRIDGE_ID. Before making this modification, thoroughly review the code to ensure compatibility with the rest of the logic and conduct thorough testing to verify that the behavior remains unchanged.