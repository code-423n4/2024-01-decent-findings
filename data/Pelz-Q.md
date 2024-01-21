# Insufficient Error Messaging in UTBOwned::onlyUtb Modifier
## Summary
The code contains a modifier named `onlyUtb` lacking a descriptive error message. A missing error message can lead to confusion and hinder efficient debugging, potentially making it challenging to identify the cause of transaction failures.

## Vulnerability Detail
The `onlyUtb` modifier employs the require statement to validate the sender's address against a predefined `utb` address. However, it lacks a clear error message, which may obscure the reason for transaction failures.

## Impact
The absence of a descriptive error message in the modifier could result in difficulties for developers and users in understanding why certain transactions fail. This may lead to delays in troubleshooting and potentially compromise the integrity of the smart contract.


## Code Snippet
reference: https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBOwned.sol#L11-L14
```solidity
modifier onlyUtb() {
    require(msg.sender == utb, "Only UTB address can call this function");
    _;
}
```
## Tool used
Manual Review

## Recommendation
I recommend to include descriptive error messages in modifiers and require statements to enhance code readability and facilitate efficient debugging. The added clarity will assist developers and users in understanding the reasons behind transaction failures, improving the overall robustness of the smart contract.