## Summary

no | File |
|-|:-|
| [[File-1](#file-1)] | UTB.sol |
| [[File-2](#file-2)] | UTBExecutor.sol | 
| [[File-3](#file-3)] | UTBFeeCollector.sol | 
| [[File-4](#file-4)] | BaseAdapter.sol | 
| [[File-5](#file-5)] | DecentBridgeAdapter.sol | 
| [[File-6](#file-6)] | StargateBridgeAdapter.sol | 
| [[[File-7](#file-7)] | SwapParams.sol | 
| [[File-8](#file-8)] | UniSwapper.sol | 
| [[File-9](#file-9)] | DcntEth.sol | 
| [[File-10](#file-10)] | DecentEthRouter.sol | 
| [[File-11](#file-11)] | DecentBridgeExecutor.sol | 


## Analysis Issue Report 


### [File-1]

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol




#### Admin Abuse Risks:

1-  The contract has an `onlyOwner` modifier on several critical functions, which restricts access to the owner.
This helps mitigate admin abuse risks as only the owner can modify critical parameters such as the executor, wrapped token, and fee collector.

2-  The `registerSwapper` and `registerBridge` functions are also restricted to the owner, preventing unauthorized registration of swappers and bridge adapters.



#### Systemic Risks:

The contract uses an external `executor` and `feeCollector`.
The risk here lies in the correct implementation and security of these external dependencies.
If compromised, it could affect the overall system's integrity.

#### Technical Risks:

1-  The contract uses low-level calls such as `wrapped.deposit{value: swapParams.amountIn}();` and `executor.execute{value: amountOut}()`.
Ensure these calls are secure and well-audited to prevent reentrancy attacks or unexpected behavior.

1-  The contract relies on external dependencies (`ISwapper` and `IBridgeAdapter`).
Ensure these interfaces are well-defined and that interacting contracts adhere to these standards.


#### Integration Risks:

The contract integrates with external swappers and bridge adapters.
Ensure that these external contracts are well-audited, and their interfaces match the expectations of this contract.


#### Non-Standard Token Risks:

The contract interacts with ERC-20 tokens.
Ensure that these tokens adhere to the ERC-20 standard.
Non-standard or malicious tokens may pose risks.


#### Overall:

1-  The contract seems to follow good practices by using modifiers to restrict access and separating concerns into different functions.

2-  It's important to verify the correctness and security of external dependencies (`executor`, `feeCollector`, `ISwapper`, `IBridgeAdapter`) to ensure the overall system's robustness.

3-  Consider adding more comments to explain complex logic and ensure readability for future developers or auditors.

4-  Perform thorough testing, especially with edge cases, to ensure the reliability of the contract.


### [File-2]

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol




#### Admin Abuse Risks:

The contract includes the `onlyOwner` modifier on both `execute` functions, limiting access to the owner.
This helps mitigate admin abuse risks as only the owner can execute payment transactions.



#### Systemic Risks:

The contract relies on an external dependency (`IERC20`), and the correct implementation of this interface is crucial for the contract's functionality.
Ensure that the external token contract behaves as expected.

#### Technical Risks:

1-  The use of low-level calls, such as `target.call{value: amount}`(`payload`), can be risky.
Ensure that reentrancy and other security concerns are appropriately handled, as these calls might interact with untrusted or malicious contracts.
2-  The contract uses the `payable` modifier on the `refund` address, indicating that it expects the recipient to handle native token transfers.
Ensure that the recipient is a contract capable of receiving payments.


#### Integration Risks:

The contract interacts with external contracts (`target` and `paymentOperator`) assuming they adhere to certain standards.
Ensure that these contracts are well-audited, and their interfaces match the expectations of this contract.


#### Non-Standard Token Risks:

The contract interacts with ERC-20 tokens. Ensure that these tokens adhere to the ERC-20 standard.
Non-standard or malicious tokens may pose risks.


#### Overall:

The contract follows a pattern of allowing the owner to execute payment transactions with both native and ERC-20 tokens.
The conditional execution of `target.call{value: extraNative}(payload)` might lead to unexpected behavior if not carefully implemented.
Ensure that the contract calling target can handle both the extra native value and the payload.


### [File-3]

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol




#### Admin Abuse Risks:

The contract includes the `onlyOwner` modifier on critical functions, such as `setSigner` and `redeemFees`. 
This limits access to administrative functions, reducing the risk of abuse.



#### Systemic Risks:

The contract relies on external dependencies (`UTBOwned` and `IERC20`).
Ensure that these dependencies are well-audited and behave as expected.


#### Technical Risks:

1-  The use of ECDSA signatures for fee verification is implemented.
Care must be taken to ensure that the signature verification process is secure and resistant to possible attacks.

2-  The `splitSignature` function performs low-level assembly operations.
While it's a common practice, it introduces a level of complexity and potential risk.
Ensure that it is implemented correctly.

3-  The `keccak256` hash of `packedInfo` is used as part of the signature verification.
Ensure that the hashing process aligns with the expected data format and doesn't introduce vulnerabilities.


#### Integration Risks:

The contract interacts with external contracts (`UTBOwned` and `IERC20`).
Ensure that these contracts adhere to expected standards and are secure.


#### Non-Standard Token Risks:

The contract interacts with ERC-20 tokens. Ensure that these tokens adhere to the ERC-20 standard.
Non-standard or malicious tokens may pose risks.


#### Overall:

1-  The contract serves as a fee collector and verifier, ensuring that the collected fees match the expected structure and are signed by a specific address `signer`.
2-  The use of signatures adds an extra layer of security to fee collection.
3-  The redemption of fees is handled securely, with the ability to redeem fees in both native currency and ERC-20 tokens.
4-  The contract provides flexibility by allowing the owner to set the signer address.


### [File-4]

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol




#### Admin Abuse Risks:

The contract includes the `onlyOwner` modifier on the `setBridgeExecutor` function, limiting access to administrative functions to the owner.
This helps mitigate admin abuse risks by controlling the assignment of the bridge executor.



#### Systemic Risks:

The contract relies on an external dependency `UTBOwned`.
Ensure that this dependency is well-audited and behaves as expected.

#### Technical Risks:

1-  The `onlyExecutor` modifier restricts access to functions, ensuring that only the designated bridge executor can call certain functions. However, the correctness of this mechanism relies on proper configuration and management of the bridge executor address.
2-  There is no validation on the bridge executor address itself. Ensure that the address provided to `setBridgeExecutor` is valid and represents a trusted entity.


#### Integration Risks:

The contract may be part of a broader system that involves a bridge executor.
Ensure that the integration with the rest of the system is well-defined and secure.


#### Non-Standard Token Risks:

The contract doesn't interact with tokens, and there is no scope for non-standard token risks in this specific contract.


#### Overall:

1-  The contract seems to serve as a base adapter, providing a mechanism to set and control the bridge executor address.
2-  It enforces that only the owner can set the bridge executor, reducing the risk of unauthorized changes.


### [File-5]

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol




#### Admin Abuse Risks:

The contract includes the `onlyOwner` modifier on functions that modify critical parameters (`setRouter`, `registerRemoteBridgeAdapter`).
This helps mitigate admin abuse risks by ensuring that only the owner can make these changes.



#### Systemic Risks:

The contract relies on external dependencies, such as the `BaseAdapter` contract, and interfaces (`IDecentEthRouter`, `IUTB`).
Ensure these dependencies are well-audited and behave as expected.

#### Technical Risks:

1-  The `estimateFees` function calculates fees based on external calls to the `router`. 
Ensure that the `router` behaves predictably, and the calculation is accurate.

2-  Gas-related configurations (`gasIsEth`and `bridgePayload` usage) may introduce complexities.
Ensure these configurations align with the intended behavior and are well-documented.


#### Integration Risks:

1-  The contract interfaces with other components, such as the `router` and the `IUTB` interface. 
Ensure that the integration is well-defined and secure.

2-  The `receiveFromBridge` function depends on the external IUTB interface.
Verify that the interaction conforms to the expected behavior.


#### Non-Standard Token Risks:

1-  The contract interacts with tokens (`IERC20`). Ensure that the token interactions follow best practices to avoid vulnerabilities.
2-  The `bridgeToken` variable introduces a specific token. Ensure that its behavior aligns with expectations.


#### Overall:

1-  The contract seems to be an adapter for the Decent Bridge, allowing the UTB system to interact with it.

2-  The `bridge`1 function is a crucial part, performing bridging operations with payload and involving external calls. Ensure it's well-tested and secure.


3-  Verify that the LZ ID lookup and chain ID lookup mappings are correctly maintained for proper cross-chain functionality.

4-  The contract checks if `destinationBridgeAdapter[dstChainId]` is set before proceeding with the `bridge` function.

5-  Gas-related configurations need careful consideration, especially regarding the use of `msg.value` and `gasIsEth`.


### [File-6]

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol




#### Admin Abuse Risks:

The contract includes the `onlyOwner` modifier on critical functions (`setRouter`, `setStargateEth`, `registerRemoteBridgeAdapter`).
This mitigates admin abuse risks by restricting access to the owner.



#### Systemic Risks:

The contract relies on external dependencies, such as the `BaseAdapter` contract, `IStargateRouter`, and `IStargateReceiver`.
Ensure these dependencies are well-audited and function as expected.

#### Technical Risks:

1-  The contract interacts with Stargate-specific structures (`LzBridgeData` and `IStargateRouter.lzTxObj`).
Ensure that these structures are well-defined and that their usage aligns with the Stargate protocol.

2-  Gas-related calculations and usage of `msg.value` need careful consideration.
Ensure gas calculations are accurate, and the use of `msg.value` aligns with the intended behavior.


#### Integration Risks:

1-  The contract interfaces with Stargate-specific functions (`sgReceive`).
Verify that the integration conforms to the expected behavior of the Stargate protocol.

2-  The `bridge` function performs critical bridging operations and involves external calls.
Ensure this function is well-tested and secure.

3-  The contract uses a variety of parameters from the `additionalArgs` argument. Ensure proper decoding and usage of these parameters.


#### Non-Standard Token Risks:

1-  The contract interacts with tokens (`IERC20`). Ensure that token interactions follow best practices to avoid vulnerabilities.

2-  The `getBridgeToken` and `getBridgedAmount` functions handle token-related logic. Verify that these functions behave as intended.


#### Overall:

1-  The contract seems to be an adapter for the Stargate protocol, allowing the UTB system to interact with it.

2-  The `bridge` function interacts with the Stargate router and performs a token transfer. Ensure that the token transfer and router interactions are secure.

3-  The `sgReceive` function handles the reception of tokens from Stargate. Ensure that it processes incoming tokens correctly and securely.

4-  The `getValue`, `getLzTxObj`, `getDstChainId`, `getSrcPoolId`, `getDstPoolId`, and `getDestAdapter` functions provide utility and are used internally for various purposes.


### [File-7]

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol



#### Admin Abuse Risks:

There are no admin-related functions or modifiers in the provided code. However, if this library and struct are used in other contracts, their usage in those contracts should be analyzed for potential admin abuse.

#### Systemic Risks:

The library and struct are simple and don't seem to introduce systemic risks on their own. However, if they are used in a broader system, ensure that the interactions within that system are well-defined and secure.

#### Technical Risks:

1-  The `SwapDirection` library defines two constants (`EXACT_IN` and `EXACT_OUT`). If used incorrectly, such as swapping in the wrong direction, it could result in unexpected behavior. Ensure that developers using this library understand its intended use.

2-  The `SwapParams` struct is well-structured and clear. However, it's important to ensure that the `path`field, which likely represents the swapping path, is constructed and used correctly.


#### Integration Risks:

If this library and struct are integrated into other contracts, the integration should be analyzed. Ensure that the contracts using these components adhere to the intended logic.


#### Non-Standard Token Risks:

The `SwapParams`struct interacts with ERC-20 tokens (`tokenIn` and `tokenOut`). If these tokens are non-standard or have specific characteristics, ensure that the usage complies with the respective token standards and best practices.


#### Overall:

1-  This code snippet seems to be a part of a larger system related to token swapping.
2-  Ensure that the usage of `SwapDirection` and `SwapParams` in the broader system is well-documented and understood by developers.
3-  If this code is part of a decentralized exchange or a similar system, thorough testing and auditing of the entire system are crucial to ensure the security and correctness of token swapping operations.


### [File-8]

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol




#### Admin Abuse Risks:

The `UniSwapper` contract inherits from `UTBOwned`, and ownership is restricted to the contract deployer (`onlyOwner` modifier). Admin abuse risks are limited to the actions allowed by the owner, such as setting the router and wrapped token.



#### Systemic Risks:

The `UniSwapper` relies on the external contract `IV3SwapRouter for Uniswap V3 functionality. Systemic risks may arise if there are vulnerabilities in the external contract or if the contract's behavior changes unexpectedly.


#### Technical Risks:

1-  The `SwapParams` and `SwapDirection` libraries are included and used for structuring swap parameters. If these libraries are used correctly across the system, there are minimal technical risks associated.

2-  The `_receiveAndWrapIfNeeded` function handles the deposit of Ether into the contract and the transfer of ERC-20 tokens. The logic seems appropriate, but ensure the correctness of token handling and Ether wrapping.


#### Integration Risks:

Integration risks may arise if the external contracts (`IV3SwapRouter`, `IERC20`, `IWETH`) are not compatible or if their behavior changes. 
Proper integration testing and version compatibility checks are crucial.


#### Non-Standard Token Risks:

The contract interacts with ERC-20 tokens and wrapped Ether (`IWETH`). Ensure that these tokens adhere to the respective standards and follow best practices. 
Any non-standard behavior should be handled appropriately.


#### Overall:

1-  The contract facilitates swapping using Uniswap V3 functionality.

2-  The owner has control over critical parameters such as the Uniswap router and the wrapped token.

3-  Ensure that the Uniswap V3 router is deployed and properly configured before using this contract.

4-  Integration with external contracts should be carefully validated, and versioning considerations should be taken into account.

5-  Consider additional security measures, such as implementing reentrancy guards, if they are not handled in external contracts.

### [File-9]

### The bellow issues related to the File ( Link )
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol




#### Admin Abuse Risks:

1-  The `DcntEth` contract inherits from `OFTV2` and allows minting and burning of tokens.

2-  Admin abuse risks include the potential misuse of mint and burn functions. 
The `onlyRouter` modifier limits certain functions to the router, reducing admin abuse risks.



#### Systemic Risks:

Systemic risks are relatively low as the contract is an extension of `OFTV2`. Risks are dependent on the functionality and security of the underlying `OFTV2` contract.
Ensure that `OFTV2` is well-audited and secure.

#### Technical Risks:

The `setRouter` function allows the owner to set the router address.
Ensure that only authorized and secure routers are set to prevent potential attacks.


#### Integration Risks:

Integration risks may arise if the router integration is not properly implemented or if the router itself has vulnerabilities. Ensure that the router contract is reliable and well-tested.


#### Non-Standard Token Risks:

The token (`DcntEth`) is derived from `OFTV2`, and its behavior should align with the standard ERC-20 token.
Non-standard token risks may arise if the underlying `OFTV2` has non-standard behavior or vulnerabilities.


#### Overall:

1-  The contract is a specific implementation of an ERC-20 token (`DcntEth)`.

2-  Admin abuse risks are somewhat mitigated by the `onlyRouter` modifier, limiting certain functions to the router.

3-  Ensure that the router is securely implemented and authorized to interact with this contract.

4-  Technical risks are relatively low, but thorough testing and auditing are recommended.

5-  Integration risks are dependent on the reliability and security of the router contract.

6-  Non-standard token risks may be present if the underlying `OFTV2` has non-standard behavior.


### [File-10]

### The bellow issues related to the File ( Link )
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol




#### Admin Abuse Risks:

1-  Admin abuse risks are relatively low due to the presence of the `onlyOwner` modifier in critical functions such as `registerDcntEth`, `addDestinationBridge`, and others.

2-  The contract owner can set the `dcntEth` contract and add destination bridges, which can potentially influence the behavior of the contract.



#### Systemic Risks:

1-  Systemic risks are moderate and depend on the behavior and security of the `dcntEth` and other interfaces utilized by the contract.

2-  The contract interacts with external contracts such as `IWETH`, `IDcntEth`, `IOFTReceiverV2`, `IDecentBridgeExecutor`, and others. Ensure these contracts are secure and well-audited.

#### Technical Risks:

1-  Technical risks include potential vulnerabilities in the implementation of `_bridgeWithPayload`, `estimateSendAndCallFee`, and other critical functions.

2-  The `onlyEthChain` and o`nlyLzApp` modifiers ensure certain functions are called only in specific contexts, reducing technical risks.


#### Integration Risks:

1-  Integration risks are present due to the reliance on external contracts. Ensure proper integration and compatibility with the specified interfaces.

2-  The contract interacts with the `IWETH`, `IDcntEth`, and other contracts. Verify the correct configuration of these contracts to prevent unexpected behavior.


#### Non-Standard Token Risks:

1-  Non-standard token risks are minimal as the contract primarily deals with WETH and DcntEth, which are expected to adhere to standard ERC-20 interfaces.

2-  However, thorough testing and verification of the behavior of these tokens are necessary.


#### Overall:

1-  The contract serves as a router for interacting with DcntEth and other bridges.

2-  Admin abuse risks are mitigated by the use of the `onlyOwner` modifier in critical functions.

3-  Ensure the secure configuration of external contracts, especially `IWETH`, `IDcntEth`, and `IDecentBridgeExecutor`.

4-  Thoroughly test and audit the critical functions, especially those dealing with bridge operations and fee estimations.

5-  Integration risks are present due to the reliance on external contracts; ensure proper integration and compatibility.

6-  The contract contains modifiers such as `onlyEthChain` and `onlyLzApp `to enforce specific conditions for function execution.



### [File-11]

### The bellow issues related to the File ( Link )
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol




#### Admin Abuse Risks:

1-  Admin abuse risks are limited due to the `onlyOwne`r modifier on the `execute` function. This ensures that only the owner can trigger the execution of transactions.

2-  However, the potential abuse risk lies in the owner's ability to configure the contract and set the WETH contract address and gas currency type.



#### Systemic Risks:

1-  Systemic risks are relatively low as the contract primarily interacts with the `WETH` contract and handles the execution of transactions based on whether the gas currency is ETH or not.

2-  Ensure that the WETH contract is secure and well-audited, as it directly impacts the behavior of the `DecentBridgeExecutor`.

#### Technical Risks:

1-  Technical risks include potential vulnerabilities in the implementation of the `_executeWeth` and `_executeEth` helper functions.

2-  The reliance on external calls and the execution of arbitrary payloads may introduce security risks. Ensure that the target contracts are trusted and secure.


#### Integration Risks:

1-  Integration risks are minimal as the contract mainly interfaces with the WETH contract. Ensure proper configuration of the WETH contract address.

2-  The contract does not heavily rely on external integrations, reducing the overall integration risks.


#### Non-Standard Token Risks:

Non-standard token risks are not applicable in this context as the contract primarily deals with WETH, which is expected to adhere to the standard ERC-20 interface.


#### Overall:

1-  The contract serves as an executor for interacting with WETH and handling the execution of transactions.

2-  Admin abuse risks are mitigated by the use of the `onlyOwner` modifier on critical functions.

3-  The contract's behavior depends on the security of the WETH contract, so ensure that the WETH contract is secure and well-audited.

4-  Thoroughly test and audit the helper functions `_executeWeth` and `_executeEth` to ensure the secure execution of transactions.

6-  The contract handles Ether reception through the `receive` and `fallback` functions.

### Time spent:
11 hours