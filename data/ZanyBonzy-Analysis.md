#  **Advanced Analysis Report for Curves** 

[Audit approach](#1-audit-approach)

[Scope and Architecture Overview](#2-scope-and-architecture-overview)

[Codebase Overview](#3-codebase-overview)

[Centralization Risks](#4-centralization-risks)

[Systemic Risks](#5-systemic-risks)

[Recommendations](#6-recommendations)

[Conclusions](#7-conclusions)

[Resources](#8-resources)


## **1. Audit approach**

**Protocol Familiarization**: The review began by analyzing the protocol documentation, including the readMe, decentxyz docs, and materials that provided insight as to how the protocol is meant to function. This was followed by a preliminary examination of the relevant contracts and their tests.

**Deep Contract Review**: A meticulous, line-by-line review of the in-scope contracts was conducted, meticulously following the expected interaction flow.

**Issue Discussions and Resolutions**: Potential pitfalls identified during the review were discussed with the developers to clarify whether they were genuine issues or intentional features of the protocol. ATtack vectors and ways to break protocol's were explored, while also ensuring that contracts concur to best programming practices.

**Report Generation**: Audit reports were finalized based on the findings.

***

## **2. Scope and Architecture Overview**


### **[UTB library](https://github.com/code-423n4/2024-01-decent/tree/main/src)**
This is the library that allows for chain transaction routing through selected bridges.

#### **[UTB.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol)**

- Users interact with this contract to execute same chain or cross chain transactions depending on their choice.

**Same chain transactions using `swapAndExecute`**: For users who are interested in performing a transaction on the same chain but with the use of a different token, e.g User with USDC wants to buy an NFT using AVAX on mainnet. The function swaps the user's initial swap for a fee and executes the transaction using the resulting token from the swap. It executes in one transaction, the swapping of tokens and execution of subsequent transaction.

**Cross chain transactions using `bridgeAndExecute`**: Users call this function when they need to execute a transaction on another chain with funds from their initial chain, e.g User with ETH on mainnet can bid on an auction using ETH or ARB on arbitrum chain. The function bridges, swaps and executes the user's transaction in one seamless transaction.

**Fee retrieval using `retrieveAndCollectFees`**: A certain fee charged for bridging and swapping transactions are retrieved using the `retrieveAndCollectFees` modifier and sent to the `UTBFeeCollector` contract, where it can be retrieved.

<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/298617059-d8487af7-ed9f-4d02-9c60-c73054ef816f.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240122%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240122T142606Z&X-Amz-Expires=300&X-Amz-Signature=075a80d501bd5906c068e9b468501e55ce60c0c2a140eb8ae279c02ab26c19bd&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="Utb"> sLOC - 232
</p>

#### **[UTBExecutor.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol)**

- This contract executes all payment transactions that the `UTB` contract needs to get done to the paymentOperator. Functions here are not callable by anyone except the Owner which is the `UTB` contract. 

<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/298617252-0f1fe66a-4710-46f6-87c2-ea1ce967ee82.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240122%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240122T142713Z&X-Amz-Expires=300&X-Amz-Signature=826d37027e8281035b82f6fa536cd2380e28775a8148e76e84eb38d12bca036b&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="UtbExecutor"> sLOC - 52
</p>

#### **[UTBFeeCollector.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol)**

- This contract receives the the charged fees from `UTB` upon which the owner can redeem it.

**Fee collection from UTB and redemption**: Through the `collectFees` function, the contract retrtieves fees from `UTB` using an ECDSA signature for verification. Afther successful fee collection, the owner can finally call the `redeemFees` to collect the fees in a specific tokens and amounts.

<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/298614004-c95ff068-ffb4-49ca-8f05-a8a44950a395.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240122%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240122T141551Z&X-Amz-Expires=300&X-Amz-Signature=80ce9736003a5eea632b185d8a09c64a9002b48503213daaa6fee469987c8e71&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="UtbFeeCollector"> sLOC - 50
</p>


### **[Decent-bridge library](https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src)
Decent is the protocol's own bridge for bridging and swapping user's tokens before executing the transactions.

#### **[DcntEth.sol](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol)**

- DcntEth is decent's implementation of `ERC20` token. It's based on the OFTV2's implementation of the ERC20 and is expected to conform to the ERC20 standard.

**Token minting and burning**: The contract allows for token minting and burning by the owner or the set decent router. 

<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/298615924-025488f6-a310-4246-9728-e909a2a77e57.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240122%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240122T142229Z&X-Amz-Expires=300&X-Amz-Signature=fcdb30c5f70afc5a9f939fc91c0d8c211e66584ef1499267e5b227df6f42bbea&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="DcntEth"> sLOC - 27
</p>


#### **[DecentEthRouter.sol](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol)**

- Holds the basic functions of the bridge. Bridge transactions to decent bridge are executed here.

**Token bridging**: Through the `bridge` and `bridgeWithPayload` functions, tokens can be transferred from one chain to a destination chain for ETH and WETH.

**ETH/WETH Liquidity addition and removal**: DcnthEth token can be minted or burned by adding or removing ETH/WETH to add or remove liquidity to the bridge. 

**ETH/WETH redemption**: If there's enough in the reserves, users can redeem their DcntEth for ETH/WETH by calling the `redeemEth` and `redeemWeth` functions


<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/298616319-eb545b59-d3e7-46f6-8ff9-4d297087cda8.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240122%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240122T142334Z&X-Amz-Expires=300&X-Amz-Signature=f89d24c3b5b823abacbcf73b24fcb9d3c0b3bb67210206c0fdd05c7e115d7638&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="DecentEthRouter"> sLOC - 290
</p>


#### **[DecentBridgeExecutor.sol](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol)**	

- Executes the user's post bridge transaction to a target upon receiving DcntEth from the router. The function is can only be called by the owner, which is the router.

<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/298616523-d473ef51-c63d-490b-8fb4-86aaf8910b6b.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240122%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240122T142417Z&X-Amz-Expires=300&X-Amz-Signature=225291176f9abbaa94c242e59117c03374997d41df72fae09359686381fe68c3&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="DecentBridgeExecutor"> sLOC - 57
</p>


### **[Bridge-Adapter contracts](https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters)**
These are the adapter implementations for Decent and Stargate bridges. 

#### **[BaseAdapter.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol)**

- The base adapter serves as the basic implementation of the adapter. It allows the owner to set the bridge executor.

<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/298614312-6c2722e9-3ea2-4978-bb85-e7423f023fdb.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240122%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240122T141655Z&X-Amz-Expires=300&X-Amz-Signature=40e19c4398c589a622172c2376666e2de1a750473e0c3cac3a7f2d940d09736c&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="BaseAdapter"> sLOC - 16
</p>


#### **[DecentBridgeAdapter.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol)**

- This relays and receives information from Decent bridge when users call the `bridgeAndExecute` transactions. Payload, swap and transaction parameters are relayed from UTB to the Decent router. It also implements `receiveFromBridge` function to receive transaction information and tokens from Decent bridge. 


<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/298614873-2daca650-b84c-4d92-8b2e-bc9ef86576f8.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240122%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240122T141848Z&X-Amz-Expires=300&X-Amz-Signature=813e9563552609e2b22301eb72e719abe36b2e75cb83afa02a2f26d94b7200a6&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="DecentBridgeAdapter"> sLOC - 137
</p>

#### **[StargateBridgeAdapter.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol)**	

- This functions like the `DecentBridgeAdapter` but with StarGate instead of Decent bridge. As stipulated by stargate docs, the contract implements `sgReceive` function to receive tokens and transaction payload from Stargate.

<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/298615109-feafe122-a69e-4017-bd1e-d86d278e5108.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240122%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240122T141937Z&X-Amz-Expires=300&X-Amz-Signature=83bc8ddb378a2a57d3f7b52c405cc47793271e2f78f4436a143d523d2cd935cf&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="StargateBridgeAdapter"> sLOC - 190
</p>


### **[Swapper contracts](https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers)**	

These contracts hold information about user swap transactions, and also execute the swaps.

#### **[SwapParams.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol)**

- This is a library, struct that holds the swap parameters and direction.  


#### **[UniSwapper.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol)**

- This contract uses uniswap's IV3SwapRouter to execute swap transactions. It also includes functions to refund users in cases where more than needed tokens are sent.

<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/298615658-a39de3a4-7bfe-4641-a157-c47a9f47ebf5.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240122%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240122T142127Z&X-Amz-Expires=300&X-Amz-Signature=c6dfdcf74d3b43356ca8b8c0508289807be2f3ef1c2d82f69f0a9572866c2710&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="UniSwapper"> sLOC - 145
</p>

***
## **3. Codebase Overview**

**Audit Information** - For the purpose of the security review, the Decent Protocol consists of eleven smart contracts in scope totaling 1193 SLoC. Its core design principle is composition, enabling efficient and flexible integration. It is scheduled to be deployed on the chains supported by layerzero which includes mainnet, arbitrum, optimism etc, giving each user the freedom to transact in and across multiple chains. A knowledge of layerzero is needed to understand the protocol.

**Documentation and NatSpec** - The codebase is divided into two parts. The UTB implementation (contracts outside the lib folder) and the Decent library. The UTB implementation has no official documentation, just the provided c4 readme. The Decent library has a well written documentation on the other hand. Certain contracts e.g UTB.sol are well commented (not necessarily to NatSpec), some however, e.g DecentBridgeAdapter.sol are not commented at all. As can be expected, t    his made the audit a bit more challinging.

**Testability** - The overall test coverage is about 75% which is not bad. Some of the contracts also have coverages on the lower end. There are no invariant or fuzz tests implemented.

**Gas Optimization** - The codebase isn't badly optimized for gas usage. Although, certain basic gas saving techniques are ignored. Via-ir is not enabled for deployment, require and revert strings were used, important state variables that set in the constructors are declared as public and so on.

**Error handling and Event Emission** -  Require revert errors were used in the codebase in place of custom errors. Events are well handled, emitted when needed for important transactions and parameter changes.

Other basic safety measures, two step address and state variable updates, protected fallback/receive functions, zero address checks etc were also not implemented.

***

## **4. Centralization Risks** 
As with any protocol that incorporates admin functions, the protocol is also vulnerable to actions of a malicious admin. Some of them include:
 - Excessive minting and burning of the DcntEth tokens through the `mintByOwner` and `burnByOwner` functions. As there's no max token supply in place, compromised or malicious owners can inflate available token supply making the tokens worthless.
 - The owner can set invalid, compromised router, executor addresses or even disable them by setting them to zero addresses (as there are no zero address checks). This puts the protocol at the mercy of a malicious or compromised owner.

***
## **5. Systemic Risks** 

- The system does not impose any caps on the number of DcntEth that can be minted, potentially leading to scalability challenges, or inflation risks.
- There are no timelocks or pause functions in place in case of emergency conditions, black swan events, and changes to system parameters pretty much take effect immediately.
- Dependency on External Protocols: Reliance on external bridges such as Stargate, Decent and Uniswap introduces potential risks.
- Limited Non-Standard ERC20 Token Support: The protocol may not support all non-standard ERC20 tokens, potentially affecting certain functions.

***
## **6. Recommendations**

- The codebase should be sanitized, and the contracts without comments should be commented to make it easier to understand, audit and use. Those with comments should also be brought up to date to conform with NatSpec. For the UTB contracts, a well written gitbook document is also needed. And the decent docs updated.

- Testing should be improved from 75% coverage, including invariant and fuzzing tests. This helps to catch basic bugs, improve modularity.

- Safe ERC20 libraries should be imported to help with token approvals and transfers, This protects from erc20 basic approval and transfer issues. Balance checks before and after transfer for tokens that charges fee on transfer. Approvals to zero for tokens like USDT, adding non-reentrant modifiers for tokens with transfer hooks and so on. This [repo](https://github.com/d-xo/weird-erc20) can be a great help for these token types.

- Timelocks and two step variable and address updates should be implemented. Most of the setter functions implement the changes almost immediately which can be jarring for most contracts/users. Adding these fixes can help protect from unexepected behaviour and give enough to make last minute executions before changes are made.

- Restrict Router Setting: Add an onlyOwner modifier to the setRouter function in the DcntEth contract to make it inaccessible to any malicious user who can set a malicious router in place of the DecentRouter contract, to mint tokens at will and block the protocol's liquidity adding and removing functionality.

- Protect Receive and Fallback Functions: Add modifiers to the receive and fallback functions to prevent unintended ETH transfers.

- Replace Require/Revert Errors: Replace require and revert statements with custom error handling to reduce gas usage

- Use Via-IR for Deployment: Utilize Via-IR for deployment to improve code transparency and audibility.
***
## **7. Conclusions**
- The protocol aims to be a stop shop for multi chain transactions in one click, through which users can execute transactions on different chains with difference source tokens without having to go through the individual token bridging, token swapping and transaction executing process. It facilitates these using Uniswap, Decent bridge and stargate bridges with options to add others in the future.

-  In conclusion, the codebase appears to be well designed which is commendable, but a number of risks were identified and they need to be fixed. Recommended measures should be implemented to protect the protocol from potential attacks. Timely audits and codebase cleanup should be conducted to keep the codebase fresh and up to date with current security times.
***
## **8. Resources**

- [C4 ReadMe](https://github.com/code-423n4/2024-01-decent/blob/main/README.md)
- [Decent.xyz docs](https://docs.decent.xyz/)
- [Stargate docs](https://stargateprotocol.gitbook.io/stargate/stargate-router-methods)



### Time spent:
12 hours