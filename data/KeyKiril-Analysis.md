## Overview Analysis on the `Decent` protocol

`Decent` facilitates seamless single-click transactions across various blockchain networks, enabling payments from any source chain or token.

`Decent` leverages two key libraries for this functionality: `UTB` and `decent-bridge`. `UTB` manages the routing of cross-chain transactions by channelling them through a selected bridge, while decent-bridge serves as Decent's custom bridge, built on top of layer zero's OFT standard.

UTB utilizes two primary functions: `swapAndExecute` and `bridgeAndExecute`. As their names imply, `swapAndExecut`e facilitates same-chain transactions for a user, allowing payments from potentially different payment tokens. On the other hand, `bridgeAndExecute` handles cross-chain transactions for a user.


## `UTB.sol` analysis

1.	Contract Inheritance and Ownership:
•	The contract inherits from the Owned contract, which implies that it has an ownership model controlled by an address.
•	The constructor initializes the owner with the deploying address.
2.	External Dependencies:
•	The contract relies on external interfaces and libraries, such as ISwapper, IUTBExecutor, IUTBFeeCollector, IWETH, and IBridgeAdapter. These interfaces and libraries provide specific functionalities.
3.	State Variables:
•	The contract has state variables such as executor, wrapped, feeCollector, swappers, and bridgeAdapters.
•	swappers and bridgeAdapters mappings are used to associate swapper and bridge adapter addresses with their respective IDs.
4.	Configuration and Setup Functions:
•	The contract provides functions (setExecutor, setWrapped, setFeeCollector) to set external dependencies and configuration parameters.
•	It has functions (registerSwapper, registerBridge) to register and map swappers and bridge adapters.
5.	Swap and Execution Logic:
•	The contract provides functions like performSwap and _swapAndExecute to perform token swaps using various swappers.
•	The swapAndExecute and bridgeAndExecute functions integrate token swaps with payment transactions, including the handling of fees.
6.	Fee Collection and Transfer:
•	The contract has a FeeStructure struct to represent fees.
•	The retrieveAndCollectFees modifier is used to transfer fees to the contract before executing the target function.
7.	Bridge Functionality:
•	The contract supports cross-chain operations using the bridgeAndExecute and callBridge functions.
•	It bridges funds, executes swaps, and handles payment transactions on the destination chain.
8.	Fallback and Receive Functions:
•	The receive and fallback functions are defined to accept incoming Ether.
9.	Modifiers:
•	The onlyOwner modifier is used to restrict certain functions to the contract owner.
10.	Security Considerations:
•	The contract involves complex interactions with external systems, including swappers and bridge adapters. Ensuring the correctness of these interactions is crucial.
•	Signature verification (retrieveAndCollectFees modifier) is used to control fee collection.
11.	Gas Limitations:
•	Some functions interact with external contracts and may require careful consideration of gas limits to avoid exceeding block gas limits.
12.	Code Comments:
•	The code contains some comments, but additional comments could improve code readability and understanding.

## `UTBExecutor` contract that facilitates the execution of payment transactions with either native currency (Ether) or ERC-20 tokens. 

1.	Inheritance and Constructor:
•	The contract inherits from the Owned contract, suggesting that it has an ownership model where certain functions can only be called by the owner.
•	The constructor initializes the owner with the deploying address.
2.	Execute Function Overloading:
•	The contract has two overloaded execute functions.
•	The first one is designed for payment transactions involving either native currency or ERC-20 tokens. It does not handle extra native fees.
•	The second one is an extended version that can handle extra native fees, which might be necessary for gas-related costs.
3.	Native Currency (Ether) Handling:
•	If the token parameter is the zero address (address(0)), it indicates native currency (Ether) is involved in the payment.
•	In such cases, the contract uses the call{value: amount}(payload) method to execute the transaction, and if it fails, it attempts to refund the sender with the same amount.
4.	ERC-20 Token Handling:
•	If the token parameter is a non-zero address, it indicates an ERC-20 token is involved in the payment.
•	The contract first transfers the specified amount of the ERC-20 token from the contract owner to itself.
•	It then approves the specified paymentOperator to spend the transferred amount.
•	The actual payment transaction is then executed using target.call(payload), and if it involves extra native fees (extraNative > 0), those fees are forwarded using target.call{value: extraNative}(payload).
5.	Refund Handling:
•	After the payment transaction, the contract checks for any remaining balance of the ERC-20 token on itself.
•	If there is a remaining balance, it transfers that balance back to the specified refund address.
6.	Error Handling:
•	The success or failure of the payment transaction is checked, and in case of failure, a refund is attempted.
7.	Security Considerations:
•	The onlyOwner modifier is applied to both execute functions, ensuring that only the owner (deployer) of the contract can execute payment transactions.
•	Handling of refunds and error checks provides a safety mechanism.
8.	Fallback Function:
•	The contract does not define a custom fallback function, relying on the default fallback function inherited from the Owned contract.
9.	Gas Considerations:
•	Gas considerations are important, especially when dealing with native currency. Gas limits should be carefully managed to avoid potential out-of-gas scenarios.
10.	Comments:
•	The code contains comments that explain the purpose of each function, which is good for code readability.
11.	Potential Improvement:
•	While the contract appears functional, additional checks and validations related to ERC-20 token interactions, such as handling non-standard tokens or checking return values from external calls, could enhance robustness.

## `UTBFeeCollector`, is designed to collect and manage fees in either native currency (Ether) or ERC-20 tokens. Here's an analysis of its key components:

1.	Inheritance and Constructor:
•	The contract inherits from the `UTBOwned` contract, indicating an ownership model where certain functions can only be called by the owner.
•	The constructor initializes the contract, calling the constructor of the UTBOwned base contract.
2.	State Variables:
•	signer: Stores the address used for fee verification through ECDSA signatures.
•	BANNER: A constant string representing the prefix for Ethereum signed messages.
3.	Setter Function:
•	setSigner: Allows the owner to set the address used for fee verification.
4.	Signature Splitting Function:
•	splitSignature: A private function to split an Ethereum signature into its components (r, s, v).
5.	Fee Collection Function:
•	collectFees: Receives fees in either native currency or ERC-20 tokens.
•	Verifies the signature of the fees using ECDSA recovery.
•	Checks if the recovered address matches the signer address.
•	Transfers the UTB fee in the specified token (if any) from the UTB contract to the fee collector.
6.	Fee Redemption Function:
•	redeemFees: Allows the owner to redeem collected fees in the specified token and amount.
•	If the token is the zero address, it indicates native currency (Ether), and the amount is transferred to the owner as Ether.
•	If the token is an ERC-20 token, it transfers the specified amount to the owner.
7.	Fallback and Receive Functions:
•	Both the fallback and receive functions are implemented to accept native currency (Ether) transfers.
8.	Security Considerations:
•	The onlyOwner modifier is applied to the redeemFees function, ensuring only the owner can redeem fees.
•	The ECDSA signature verification in the `collectFees` function helps ensure that fees are collected only when a valid signature is provided.

## Time Spent

Auditing: 20 hours
Researching: 14 hours
Overall: 34 hours

### Time spent:
34 hours