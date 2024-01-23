## Fallback Function Handling:

The contract uses external calls in both branches of the conditional `if (token == address(0))`. Ensure that the target contracts have sufficient gas to handle the external call and that they are not relying on complex logic within their `fallback` functions.

Recommendation: Assess the target contracts for potential gas-related issues and ensure they are designed to handle external calls properly.

## Handling of Extra Native Fees:

The `extraNative` parameter allows forwarding additional gas or native fees. Make sure to thoroughly validate and document the use of this parameter, as incorrect handling could lead to unexpected behaviour or vulnerabilities.

Recommendation: Clearly document the purpose and correct usage of the `extraNative` parameter, and ensure that it is used safely.

## Safe Math Operations:

Ensure that arithmetic operations are safe to avoid overflow/underflow vulnerabilities. In particular, check the subtraction operation:

Recommendation: Use safe math operations, such as those provided by the OpenZeppelin library, to prevent overflow/underflow issues.

## Consistency of Return Values:

The contract returns values only in case of failure `!success`. Consider providing consistent return values for success cases as well to allow for better error handling and communication.

Recommendation: Consider refining the return values to provide more information about the outcome of the operation.

## Visibility of State Variables: 

The `bridgeExecutor` state variable is declared as public, allowing anyone to read its value. 

Consider if this is the desired behaviour; if not, you might want to change it to internal or provide a getter function with proper access control.

## Error Message Handling: 

In the bridge function, when checking if the `destinationBridgeAdapter[dstChainId]` is set, the error message is constructed using `string.concat`. 

It is recommended to use the require statement directly without concatenating strings to keep the code concise. Additionally, it may be beneficial to include more detailed error messages to aid in debugging.

## Gas Handling: 

In the bridge function, there is a check if `!gasIsEth` that implies a choice between handling gas as ETH or not. However, the behaviour of this choice is not entirely clear from the provided context. Ensure that the gas handling logic aligns with the intended functionality.

## Fallback Function Restrictions: 

Consider adding restrictions to the `fallback` function to limit unexpected Ether transfers. You may use the revert statement to reject any Ether sent to the contract without a valid function call.

## Gas Estimation: 

The `estimateFees` function returns two fee values `nativeFee` and `zroFee`, but the purpose of `zroFee` is not clear. 

Ensure that the function documentation provides clarity on the meaning and use of both fee values.

## Security Considerations: 

Ensure that the contract logic, especially interactions with external contracts, has been thoroughly tested to avoid vulnerabilities.

## Expiration Timestamp: 

`Uniswap V3` functions such as `exactInput` and `exactOutput` may include an expiration timestamp parameter. 

Depending on your use case, you may want to include this parameter for additional security.
