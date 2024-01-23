# Analysis - Decent Contest

![Decent-Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FiLkzmJ26ue3.0&w=256&q=75)

## Description overview of The Decent Contest

Decent allows users to perform transactions across multiple blockchain networks (like Ethereum, Polygon, Optimism etc.) in a single click, even if they hold funds/tokens on a different chain.

### The protocol has two components phases:

- **UTB and Decent Bridge**.

  - UTB handles routing the cross-chain transaction and passes it to the appropriate bridge. It supports functions like swapAndExecute (for on-chain swaps) and bridgeAndExecute (for cross-chain transactions).

  - Decent Bridge is built on LayerZero's OFT standard and acts as the actual bridge, facilitating the transfer of tokens/funds from one chain to another.

### Implementation Standards:

- **Swappers**: These are modules that perform same-chain transactions. They must implement the ISwapper interface.

- **Bridge Adapters**: These are modules that facilitate cross-chain transactions. They must implement the IBridgeAdapter interface. Examples include DecentBridgeAdapter and StargateBridgeAdapter.

Decent simplifies the process of making transactions across various blockchains, allowing users to use different tokens and providing flexibility in payment methods. The platform is built on specific libraries and standards to ensure compatibility and extendability.

**The key contracts of Decent protocol for this Audit are**:

As the audit of these contracts, I checked for potential security vulnerabilities, such as reentrancy, access control issues, and logic flaws. Additionally, thoroughly test the functions and roles defined in these contracts to make sure they behave as expected.

## System Overview

### Scope

- **UTB.sol**: The `UTB` contract facilitates one-click cross-chain transactions by coordinating token swaps, fee collection, and interaction with multiple blockchain networks through swappers and bridge adapters in the Decent protocol.

- **UTBExecutor.sol**: This contract extending the Owned contract, enables the execution of payment transactions involving native or ERC20 tokens, handling `transfers`, `approvals`, and `refunds`, with flexibility for additional gas or native fees in the Decent protocol.

- **UTBFeeCollector.sol**: UTBFeeCollector contract inheriting from UTBOwned, facilitates the `collection` and `verification` of fees, supporting both `native` and `ERC20` tokens, with a specified signer for fee structure verification in the Decent protocol.

- **BaseAdapter.sol**: The `BaseAdapter` contract establishes a foundation for `bridge` adapters in the Decent protocol, providing a mechanism to set the bridge `executor` and a modifier to restrict certain functions to be callable only by the designated bridge executor.

- **DecentBridgeAdapter.sol**: Serves as a bridge adapter in the Decent protocol, facilitating token transfers between different blockchain networks using the Decent protocol, with customizable fee estimation and `bridging` functionality.

- **StargateBridgeAdapter.sol**:`This contract is a bridge adapter for the Stargate protocol, enabling cross-chain token transfers with customizable`fee estimation and `integration` with the Stargate network.

- **UniSwapper.sol**: The `UniSwapper` contract, inheriting from UTBOwned and implementing the ISwapper interface, serves as a decentralized exchange adapter for UniSwap, enabling token swaps with customizable parameters, fee estimation, and seamless integration into the Universal Token Bridge (`UTB`) ecosystem.

- **DcntEth.sol**: The contract is extending the OFTV2 token contract, represents a decentralized token on the Ethereum blockchain named "Decent Eth" (with symbol "DcntEth") that supports minting and burning functionalities, and is associated with a Decent Eth Router for controlled access to these operations.

- **DecentEthRouter.sol**: The contract is an Ethereum router facilitating cross-chain transactions with DecentEth tokens, supporting the transfer and call functionalities, and managing reserves of `WETH` for bridging operations between Ethereum and other chains.

- **DecentBridgeExecutor.sol**: This contract is serving as an executor for cross-chain transactions, handling the execution of transactions with either `WETH` or native Ether, based on the gas currency configuration, and facilitating calls to target contracts.

### Protocol Roles

Here are the key roles

1. **Owner Role:**

   - Identified by the `onlyOwner` modifier.
   - Typically has privileged access to certain functions that are critical for the protocol's owner.
   - Responsible for setting configurations, adding destination bridges, and managing the protocol.

2. **Router Role:**

   - Identified by the `onlyRouter` modifier.
   - The router is responsible for specific actions and configurations within the protocol, particularly related to routing and interacting with other components of the system.

3. **LzApp Role:**

   - Identified by the `onlyLzApp` modifier.
   - Represents the Layer Zero application, specifically the DecentEthRouter contract, allowing it to call certain functions.

4. **User Roles:**
   - Various modifiers such as `userDepositing` and `userIsWithdrawing` are used to manage user-specific actions.
   - Users can deposit and withdraw assets from the protocol.

**Privileged Roles**

Some privileged roles exercise powers over the contracts:

- **Bridge Executor**

```solidity
    modifier onlyExecutor() {
        require(
            msg.sender == address(bridgeExecutor),
            "Only bridge executor can call this"
        );
        _;
    }
```

- **Router**

```solidity
    modifier onlyRouter() {
        require(msg.sender == router);
        _;
    }
```

- **lz App**

```solidity
    modifier onlyLzApp() {
        require(address(dcntEth) == msg.sender, "DecentEthRouter: only lz App can call");
        _;
    }
```

These roles help define the permissions and access levels within the protocol, ensuring that only authorized entities can perform certain operations. The roles contribute to the overall security and functionality of the decentralized application by controlling access to critical functions and resources.

## Approach Taken-in Evaluating The Decent Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the Decent Protocol.

    I start with the following contracts, which play crucial roles in the Decent protocol:

    **Main Contracts I Looked At**

    I start with the following contracts, which play crucial roles in the Decent protocol:

                UTB.sol
                DecentBridgeAdapter.sol
                StargateBridgeAdapter.sol
                UniSwapper.sol
                DecentEthRouter.sol

    I started my analysis by examining the `UTB.sol` contract, as this contract contains the core logic for Decent, enabling one-click transactions across different blockchain networks through functions like performSwap, swapAndExecute, and bridgeAndExecute

    Then, I turned my attention to the `DecentBridgeAdapter.sol` contract a critical component of Decent, which extends the BaseAdapter and IBridgeAdapter interfaces, facilitating cross-chain transactions by interacting with the Decent Eth Router, estimating fees, and managing the bridging of funds and execution of transactions across different chains.

    The `StargateBridgeAdapter` contract is an integral part of Decent, extends the BaseAdapter, IBridgeAdapter, and IStargateReceiver interfaces, facilitating cross-chain transactions via the Stargate protocol, managing the bridging of funds and execution of transactions across different chains, estimating fees, and interacting with the Stargate Router through functions like bridge, getValue, and sgReceive.

    The `UniSwapper` contract, an essential component of the Decent protocol, extends the UTBOwned and ISwapper interfaces, providing functionality for swapping tokens using Uniswap's V3 Router, handling different swap scenarios, and interacting with the specified Uniswap router to facilitate token swaps with features like swap, swapNoPath, swapExactIn, and swapExactOut.

2.  **Documentation Review**:

    Then went to Review [This Documentation](https://docs.decent.xyz/) for a more detailed and technical explanation of Decent protocol.

3.  **Compiling code and running provided tests**:

    - To setup the repo, first run forge install + pnpm i

    - To run the tests, simply add the relevant files to your .env, referencing .env.example, then run forge test

4.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Codebase Quality

Overall, I consider the quality of the Decent protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The Decent Protocol contracts demonstrates good maintainability through modular structure, consistent naming. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.                                                                                                                                                                                                                                                                                                                   |
| **Code Comments**                        | During the audit of the Decent contracts codebase, I found that some areas lacked sufficient internal documentation to enable independent comprehension. While comments provided high-level context for certain constructs, lower-level logic flows and variables were not fully explained within the code itself. Following best practices like NatSpec could strengthen self-documentation. As an auditor without additional context, it was challenging to analyze code sections without external reference docs. To further understand implementation intent for those complex parts, referencing supplemental documentation was necessary.                             |
| **Documentation**                        | The documentation of the Decent project is quite comprehensive and detailed, providing a solid overview of how Bridging is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as diagrams, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors. |
| **Testing**                              | The audit scope of the contracts to be reviewed is 75%, with the aim of reaching 100% to increase the safety.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. but some order of functions does not follow the Solidity Style Guide According to the Solidity Style Guide, functions should be grouped according to their visibility and ordered: constructor, receive, fallback, external, public, internal, private. Within a grouping, place the view and pure functions last.                                                                                                                                                                                                                                                                                                |
| **Error**                                | Use custom errors, custom errors are available from solidity version 0.8.4. Custom errors are more easily processed in try-catch blocks, and are easier to re-use and maintain.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the Decent protocol. These risks encompass concentration risk in `UTB`, `UTBExecutor` risk and more, third-party dependency risk, and centralization risks arising.

Here's an analysis of potential systemic and centralization risks in the contracts:

### Systemic Risks:

1. **UTB.sol:**

   - External Calls Without Checks: External calls, such as `performSwap` and `callBridge`, are made without checking the return values. A failed external call might leave the contract in an inconsistent state. It's important to handle potential failures properly.

2. **UTBFeeCollector.sol**

   - Signature Verification Reliance: The security of fee collection relies heavily on the correctness of the signature verification process. Any vulnerability or weakness in the signature verification logic could lead to unauthorized fee collection.

3. **StargateBridgeAdapter.sol**

   - Value Calculation Precision: The calculation of the `value` variable in the `callBridge` function involves multiplication and division operations with potentially large numbers. Ensure that these calculations are precise and do not result in unintended rounding errors.

4. **UniSwapper.sol**

   - Single swapping mechanism relies on Uniswap router which could break or have exploits impacting all trades.

### Centralization Risks:

1.  **UTB.sol:**

    - Single Owner Control: The contract is owned by a single address (Owned), creating centralization risks. The owner has significant control over critical functions such as setting the executor, wrapped token, fee collector, registering swappers, and bridge adapters.

    - Fee Collection Centralization: The fee collection mechanism (`feeCollector.collectFees`) is central to the contract, relying on a single fee collector contract. If this fee collector becomes compromised or malfunctions, it could impact the entire fee collection process.

    - Bridge Adapter Registration: The registration of bridge adapters is centralized, allowing only the owner to add or modify bridge adapters. This creates a single point of control and potential risk if the owner becomes malicious or compromised.

2.  **UTBFeeCollector.sol**

    - Single Owner Control: The contract is owned by a single address (`UTBOwned`), creating centralization risks. The owner has significant control over critical functions, such as setting the signer and redeeming fees.

    - Signer Configuration: The signer address, crucial for fee verification, is set by the owner. If the owner is compromised or acts maliciously, they can set a malicious signer, compromising the entire fee collection process.

    - RedeemFees Function: The ability to redeem fees is centralized in the owner. This function allows the owner to withdraw collected fees, and its execution is solely dependent on the owner's decision.

3.  **DecentBridgeAdapter.sol**

    - Centralized Destination Bridge Adapter Registration: The registration of remote bridge adapters (`destinationBridgeAdapter`) is centralized and controlled by the owner. If the owner is compromised or acts maliciously, they could register a malicious destination bridge adapter.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the Decent protocol.**

## Conclusion

Here are the key points I would include in a conclusion:

- Overall the code quality and architecture of the Decent protocol contracts are good. The code is well structured, tested and follows solidity best practices.

- Some opportunities for improvement include adding more internal documentation, error handling, formal verification, and upgrading to support future changes.

- There are some Centralization risks arise from single owner control over critical settings in contracts like UTB, reliance on single fee collector/signer, and centralized bridge adapter registration.

- Systemic risks like external call vulnerabilities and value calculation precision issues could impact the protocol.

- Following best practices in security, decentralization and ongoing maintenance/audits will help ensure the long-term sustainability and success of the protocol.

- It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.


### Time spent:
15 hours