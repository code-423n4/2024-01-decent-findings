<div align="center">
  <h1> Decent Protocol: Smart Contracts Analysis </h1>
  <h5>Decent enables one-click transactions using any token across chains.</h5>
  <!-- <img src="https://www.decent.xyz/" width="200" height="200" > -->
</div>

<details> 
<summary> See Index </summary>

## Index

- [Index](#index)
- [1. What is Decent: A Comprehensive Overview](#1-what-is-decent-a-comprehensive-overview)
- [2. Diving into all the contracts provided by the protocol](#2-diving-into-all-the-contracts-provided-by-the-protocol)
  - [UTB.sol](#utbsol)
  - [UTBExecutor.sol](#utbexecutorsol)
  - [UTBFeeCollector.sol](#utbfeecollectorsol)
  - [BaseAdapter.sol](#baseadaptersol)
  - [DecentBridgeAdapter.sol](#decentbridgeadaptersol)
  - [StargateBridgeAdapter.sol](#stargatebridgeadaptersol)
  - [SwapParams.sol](#swapparamssol)
  - [UniSwapper.sol](#uniswappersol)
  - [DcntEth.sol](#dcntethsol)
  - [DecentEthRouter.sol](#decentethroutersol)
  - [DecentBridgeExecutor.sol](#decentbridgeexecutorsol)
  - [Privileged Roles](#privileged-roles)
- [3. Codebase Quality](#3-codebase-quality)
- [4. Security Approach for the Provided Contracts](#4-security-approach-for-the-provided-contracts)
- [5. Addressing Architectural \& Sistemic Risks](#5-addressing-architectural--sistemic-risks)
  - [Architectural recommendations](#architectural-recommendations)
- [6. Integrate the StatefulFuzz Test Suite as an Integral Component of Your Testing Process](#6-integrate-the-statefulfuzz-test-suite-as-an-integral-component-of-your-testing-process)
  - [Implement Your Own Stateful Fuzzing Test Suite](#implement-your-own-stateful-fuzzing-test-suite)
  - [Building Your Invariants](#building-your-invariants)
  - [Write your Handler](#write-your-handler)
- [7. Recommendations beyond the code](#7-recommendations-beyond-the-code)
- [8. Insights, learnings \& common patterns](#8-insights-learnings--common-patterns)

</details>

**The approach and steps we employed to examine the underlying code.**

Our approach to analyzing the source code of the **Decent** Protocol was to simplify the information provided by the protocol, using a variety of diagrams to visually clarify the project's key contracts and break down each important part of these contracts. This enhances understanding for developers, security researchers, and users alike. We identified the fundamental concepts and employed simpler language to explain the functionality and goals of the **Decent** Protocol. Furthermore, we organized the information logically into separate sections, each with identifying titles, to provide a clear overall picture of the subject. Our primary goal was to make the information more accessible and easy to understand.

> NOTE: The provided diagrams are based on our interpretation of the protocol. In the event that any of them is not 100% accurate but aligns with the protocol's initial concept, we invite you to reach out to us via Discord (catellatech). We are willing to share the diagrams for the project team to make any necessary modifications. If the diagrams are deemed accurate, they are fully available for the protocol to use in its documentation.

## 1. What is Decent: A Comprehensive Overview

**Decent** is a platform that enables **one-click transactions** across different blockchains. It uses **UTB** for transaction routing and **decent-bridge** for **cross-chain** functionality. Users can seamlessly perform transactions, even if their funds are on a different blockchain. The platform supports same-chain transactions **(swapAndExecute)** and **cross-chain** transactions **(bridgeAndExecute)**. Swappers handle same-chain transactions, while bridge adapters manage communication with specific bridges. **UTBExecutor** executes additional logic for **UTB**, like minting NFTs. In essence, Decent streamlines cross-chain transactions with flexibility and generality.

**Overview diagram:**
<img src="https://github.com/catellaTech/Decent/blob/main/Decent.Over.drawio.png?raw=true">

## 2. Diving into all the contracts provided by the protocol

### UTB.sol

This contract provides the infrastructure to execute one-click transactions across different blockchains using swappers and bridge adapters. It also handles the collection of fees associated with these transactions.
<img src="https://github.com/catellaTech/Decent/blob/main/Decent.UTB.drawio.png?raw=true>">

### UTBExecutor.sol

The **UTBExecutor** contract enables the secure execution of payment transactions with native or ERC-20 tokens, ensuring proper fund handling and allowing for refunds in case of failures.

<img src="https://github.com/catellaTech/Decent/blob/main/Decent.UTBExecutor.drawio.png?raw=true">

### UTBFeeCollector.sol

This contract is responsible for collecting, verifying, and managing fees associated with transactions in the protocol, whether in native **Ether** or **ERC-20** tokens. Fee verification is done through digital signatures, and the collected fees can be redeemed by the owner in the specified form and quantity.

<img src="https://github.com/catellaTech/Decent/blob/main/Decent.UTBFeeCollector.drawio.png?raw=true">

### BaseAdapter.sol

This **BaseAdapter** contract is like a starting point for making bridge adapters. Think of a bridge adapter as a helper that lets different parts talk to each other in the blockchain world. By using **UTBOwned**, it makes sure that only the person who owns it can decide who the bridge helper is (bridge executor). Also, it sets things up so that only this bridge helper can use specific functions, keeping things safe and controlled when connecting to the bridge.

<img src="https://github.com/catellaTech/Decent/blob/main/Decent.BaseAdapter.drawio.png?raw=true">

### DecentBridgeAdapter.sol

This contract makes sure the Decent protocol and other bridges can chat smoothly. It helps figure out fees, keeps track of different IDs, and does the actual work of moving things around in the blockchain.

<img src="https://github.com/catellaTech/Decent/blob/main/Decent.DecentBridgeAdapter.drawio.png?raw=true">

### StargateBridgeAdapter.sol

This contract serves as a middleman, connecting the Decent protocol with various bridge adapters in a standardized way. Its role involves handling configurations, overseeing the bridging process, and maintaining smooth communication among different parts of the blockchain system.

<img src="https://github.com/catellaTech/Decent/blob/main/Decent.StargateBridgeAdapter.drawio.png?raw=true">

### SwapParams.sol

This contract play a crucial role in reNFT system responsible for creating and verifying rental orders within a digital asset framework. The detailed functionality relies on how other contracts, like Zone, Policy, Signer, and others, are implemented and integrated into the system.

<img src="https://github.com/catellaTech/Decent/blob/main/Decent.SwapParams.drawio.png?raw=true">

### UniSwapper.sol

This contract lets you swap ERC-20 tokens and, if you want, ETH too. It uses the Uniswap V3 router for these swaps and deals with various swap situations, like when you want an exact input or an exact output. It takes care of moving funds around in the right way.

<img src="https://github.com/catellaTech/Decent/blob/main/Decent.UniSwapper.drawio.png?raw=true">

### DcntEth.sol

This contract provides a basic ERC-20 token that can be managed by a specific router. The router has control over key functions, such as token creation and burning, while the owner has certain additional privileges.

<img src="https://github.com/catellaTech/Decent/blob/main/Decent.DcntEth.drawio.png?raw=true">

### DecentEthRouter.sol

It functions as an intelligent intermediary that assists in the transfer and management of assets between **Ethereum** and the **Decent** protocol, ensuring everything runs smoothly.

<img src="https://github.com/catellaTech/Decent/blob/main/Decent.DecentEthRouter.drawio.png?raw=true">

### DecentBridgeExecutor.sol

This contract acts as a transaction executor, handling the necessary logic to interact with contracts on different blockchains and managing both WETH and Ether as needed.

<img src="https://github.com/catellaTech/Decent/blob/main/Decent.DecentBridgeExecutor.drawio.png?raw=true">

### Privileged Roles

- **DecentEthRouter**:

  - **Owner**: Can register **DcntEth** contracts and add destination bridges.
  - **Lz App**: Can call the **onOFTReceived** function to process received **OFT** transactions.

- **DcntEth**:

  - **Owner**: Can adjust the associated router and perform token creation and burning operations.
  - **Router**: Can perform specific token creation and burning operations.

- **UniSwapper**:

  - **Owner**: Can configure the Uniswap router and the wrapped contract.
  - **UTB (Uniswap Token Bridge)**: Can execute specific functions related to token swapping on Uniswap.

- **DecentBridgeExecutor**:
  - **Owner**: Can execute cross-chain transactions and adjust settings.

> Given the level of control that these `roles` possess within the system, users must place full trust in the fact that the entities overseeing these `roles` will always act correctly and in the best interest of the `system` and its `users`.

## 3. Codebase Quality

Overall, we consider the quality of the Decent protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The Decent Protocol contracts demonstrates good maintainability through modular structure, consistent naming, and efficient use of libraries. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.                                                                                                                                                                                                                                                                                          |
| **Code Comments**                        | During the audit of the Decent contracts codebase, we found that some areas lacked sufficient internal documentation to enable independent comprehension. While comments provided high-level context for certain constructs, lower-level logic flows and variables were not fully explained within the code itself. Following best practices like NatSpec could strengthen self-documentation. As an auditor without additional context, it was challenging to analyze code sections without external reference docs. To further understand implementation intent for those complex parts, referencing supplemental documentation was necessary.                                   |
| **Documentation**                        | The documentation of the Decent project is quite comprehensive and detailed, providing a solid overview of how Decent protocol is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as diagrams, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors. |
| **Testing**                              | The audit scope of the contracts to be reviewed is 75%, with the aim of reaching 100%.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |

## 4. Security Approach for the Provided Contracts

- **UTB and Derivatives (UTBOwned, UniSwapper, StargateBridgeAdapter):**

  - Contracts appear to follow security best practices, including inheritance from Ownable to manage ownership authority.
  - However, the ultimate security depends on the implementation of base contracts like Ownable and the specific implementations of each contract.

- **StargateBridgeAdapter:**

  - The **bridge** and **callBridge** methods handle token transfers. It is crucial to ensure that these methods are invoked securely, and approvals are done correctly.
  - Management of the router state variable is critical for security. Ensure that only the owner can update it and that it is correctly initialized before use.

- **SwapParams and Derivatives (UniSwapper, StargateBridgeAdapter):**

  - **SwapParams** defines information needed for swaps. Ensuring that provided values are valid and secure to prevent overflows or undue manipulations is essential.
  - The security of the router **(uniswap_router)** is critical for the security of contracts using it. Ensure that only the owner can update it.

- **DcntEth:**

  - This contract inherits from **OFTV2**, but the code for **OFTV2** is not provided. The contract's security will heavily depend on the implementation of the **OFTV2** library.
  - The **setRouter** function should be restricted to the owner to ensure that only the owner can change the router.

- **DecentEthRouter:**

  - The **executor state variable** should be securely set and only allow updates by the owner.
  - Ensure that the **bridge** function is called securely and does not allow undue manipulation of funds.
  - The **redeemWeth** and **redeemEth** functions should consider handling exceptional situations, such as when someone attempts to redeem more than they own.

- **DecentBridgeExecutor:**

  - The **\_executeWeth** and **\_executeEth** functions handle the execution of arbitrary code at the destination. It is critical to ensure that the code at the destination is secure and trusted.

## 5. Addressing Architectural & Sistemic Risks

- **The Anomalies of ERC-20 Tokens:**

  - ERC-20 Tokens come in all shapes and sizes. Here's a glimpse into some of the variants and potential problems that lurk in the shadows:

    - **Missing return values:** Some tokens don’t return a boolean on ERC-20 methods. For transactions requiring a status check, this can be a potent problem.
    - **Fee on transfer:** Some tokens sneak in a fee on every transfer while others can start doing so in the future.
    - **Upgradable tokens:** These tokens, like USDC, could morph into anything over time.
    - **Rebasing tokens:** These tokens magic away your balance by meddling with different contracts.
    - **Tokens with blocklists:** Some tokens put restrictions on certain transacting parties.
    - **Low/high decimals:** Token numbers can go from unusually low to abnormally high, causing calculation mishaps.
    - **Multiple token addresses:** These tokens exist in more than one places at on
    - **Risk:** Protocol is expected to interact with any erc20 with dex liquidity, as it can be potential payment token for **swapAndExecute** or **bridgeAndExecute**

- **Centralized Ownership:**

  - Contracts like **UTBOwned**, **DecentEthRouter**, and **DecentBridgeExecutor** have centralized ownership models where a single address has ultimate control.
  - **Risk**: If the owner's private key is compromised, it could lead to unauthorized access and manipulation of critical contract functionalities.

- **Dependency on External Contracts:**

  - Contracts like **StargateBridgeAdapter** and **UniSwapper** depend on external routers (router and uniswap_router).
    **Risk**: Dependency on external contracts introduces risks such as vulnerabilities in external contracts, changes in external contract behavior, or dependency on unverified code.

- **Missing Implementation Details:**

  - Contracts like **OFTV2** and its derivatives (**DcntEth**) inherit from external contracts whose implementation details are not provided.
  - **Risk:** The security of these contracts depends on the correctness and security of the external contracts, which are not visible in the provided code.

- **Insufficient Validation in Bridge Contracts:**

  - The **StargateBridgeAdapter** and **DecentEthRouter** contracts perform **cross-chain token transfers**.
  - **Risk:** Insufficient validation or handling of unexpected states during **cross-chain** interactions could lead to fund losses or contract vulnerabilities.

- **Gas Currency Assumption:**

  - The **DecentEthRouter** assumes that gas currency is **ETH** (**gasCurrencyIsEth**). This assumption might not be valid for all chains.
  - **Risk:** Incompatibility with chains that don't use **ETH** as gas currency could lead to unexpected behaviors.

- **Limited Error Handling in Bridge Execution:**

  - In **DecentBridgeExecutor**, the execute function may transfer funds and execute arbitrary code at the destination.
  - **Risk:** Limited error handling in the execution of arbitrary code could lead to unexpected outcomes, and there's a risk of funds getting stuck or lost.

- **Insecure Calls in Bridge Contracts:**

  - The **StargateBridgeAdapter** and **DecentEthRouter** contracts use external calls to other contracts, such as the **router** and **dcntEth**.
  - **Risk:** External calls are potential security risks. Ensure these calls are secure, and reentrancy and other vulnerabilities are mitigated.

- **Assumptions about Wrapped Ether (WETH) Usage:**

  - Several contracts, including **UniSwapper** and **DecentEthRouter**, assume the use of **Wrapped Ether (weth)**.
  - **Risk:** If assumptions about **WETH** behavior or usage are incorrect, it might lead to contract vulnerabilities or unexpected outcomes.

- **Incomplete Swap Parameter Validation:**

  - The **UniSwapper** contract may perform swaps based on provided **SwapParams**.
  - **Risk:** Incomplete validation or incorrect usage of swap parameters could lead to unexpected behaviors in decentralized exchanges.

### Architectural recommendations

💡 A robust model is indispensable for systematically evaluating project risks. One highly recommended approach involves employing effective risk modeling techniques. https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

💡 We suggest establishing a foundational architecture with a **System.sol** contract, serving as a central registry where all contracts are registered. The entire system can be organized around a singular contract, such as **SystemRegistry**. This contract acts as a cohesive entity that interconnects all other contracts, allowing for a comprehensive listing of all contracts within the system. Essentially, it functions as a registry, providing a centralized point to access information about all contracts in the system..

💡Evaluate the gas consumption of each function across different scenarios during testing to ensure optimal efficiency and identify potential areas for optimization.

💡Consider decentralized ownership models or implement multi-signature control for critical functions.

💡Ensure proper validation and handling of cross-chain interactions.

## 6. Integrate the StatefulFuzz Test Suite as an Integral Component of Your Testing Process

**Some attack ideas can be tested as invariants, for example:**

- An attacker might attempt to manipulate calldata maliciously to execute unauthorized actions. To prevent this, the protocol should properly validate and authenticate transactions, ensuring that only permitted and user-authorized actions are executed.
- An attacker might attempt to retain funds in the contracts or manipulate approvals improperly. To maintain this invariant, the protocol should ensure that all funds are handled correctly, delivered to the appropriate recipients, and refunds are issued when necessary.
- An attack could involve manipulating destination transactions to revert them and endanger user funds. To prevent this, the protocol should implement robust exception handling mechanisms and ensure that users receive appropriate refunds in case of destination chain failures.

> With an understanding of these attack vectors, the protocol must implement more robust test suites.

### Implement Your Own Stateful Fuzzing Test Suite

- Create a new folder called "**invariant**" in your working environment. In this folder, you will house two **Solidity (.sol)** files named "**invariant.t.sol**" and "**handler.t.sol**" respectively. Once this setup is complete, you can commence writing your tests.

### Building Your Invariants

Start by crafting "**invariant.t.sol**," where you define your tests by laying the foundation for the 'invariant.'

### Write your Handler

Now, let's focus on creating your **handler.t.sol** file. This contract serves as an intermediary to control the contract that captures the state of the Fuzz stateful.

- Through the **handler**, you instruct your Foundry and the **Stateful Fuzzing test suite** to align correctly with the contract capturing the state of the Fuzz stateful. Essentially, you provide guidelines to **Foundry** on when to call specific functions for testing, all with precise instructions to prevent excessive function calls.

- Once your tests are configured, the next step is to write the "**handler.sol**" While you could embed assertions directly into your invariants, a cleaner approach is to compute them in the "**handler.sol**" This ensures that your assertions condense into a single line, maintaining logic validity regardless of variable input parameters. In the development of more intricate software or systems, invariants play a pivotal role in ensuring correctness.

## 7. Recommendations beyond the code

Attackers are becoming increasingly sophisticated, targeting protocols and their users not only through code vulnerabilities but also exploiting their social media presence, such as **X**. In these times more than ever, it is crucial to maintain robust layers of security. To enhance project security, it is imperative to establish control policies mandating **Two-Factor Authentication (2FA)** for all staff accounts and ensuring its widespread utilization whenever possible. It's important to acknowledge that achieving 100% foolproof security solely through smart contract codes is unattainable. To fortify security measures, consider implementing a more comprehensive policy that advocates for the use of **non-SMS 2FA methods**. For additional information, you can explore the recent **Simswap attack** on **Code4rena** detailed in this [Article](https://medium.com/code4rena/code4rena-twitter-x-incident-8b7f308a555d).

## 8. Insights, learnings & common patterns

The contracts showcase a complex architecture with diverse functionalities, including **Uniswap swaps**, **cross-chain interoperability**, and **token minting/burning**. Noteworthy is the integration with **Uniswap V3** and the handling of **Wrapped Ether (WETH)**. The cross-chain interoperability, especially through the **DecentEthRouter** contract, suggests the need for careful handling to prevent fund loss in case of failures on the destination chain. The contracts implement security mechanisms, such as swap path validation and authorization control, while the **DecentBridgeExecutor** contract takes care of executing actions in other contracts. There is proper use of common patterns like approvals and authorizations.

### Time spent:
17 hours