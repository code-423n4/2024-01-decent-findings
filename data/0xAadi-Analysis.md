# Overview

Decent simplifies cross-chain transactions with a seamless one-click process, enabling users to execute transactions on any blockchain using funds from any source chain or token. Decent leverages the LayerZero implementation of a bridge known as Decent Bridge and integrates with the Stargate Protocol for efficient interchain communications.

## Contracts

The pivotal contracts in this Audit for the protocol are:

### Core UTB Contracts
- src/UTB.sol
- src/UTBExecutor.sol
- src/UTBFeeCollector.sol

### Bridge Adapters
- src/bridge_adapters/BaseAdapter.sol
- src/bridge_adapters/DecentBridgeAdapter.sol
- src/bridge_adapters/StargateBridgeAdapter.sol

### Swapper Contracts
- src/swappers/SwapParams.sol
- src/swappers/UniSwapper.sol

### Decent Bridge (LayerZero)
- lib/decent-bridge/src/DcntEth.sol
- lib/decent-bridge/src/DecentEthRouter.sol
- lib/decent-bridge/src/DecentBridgeExecutor.sol

## Architecture

![Decent Protocol Diagram](https://gist.githubusercontent.com/AnandkKumaran/9eeeb398b954d8e0e53db8f04399f61f/raw/0c6a0d870e29433253d9ae7fc3d8ebfc11e5372e/decent3.png)

### UTB Module

The UTB Module serves as the implementation hub for core business logic, offering accessible functions to ordinary users. It empowers users to seamlessly swap currencies from the incoming to the outgoing token, bridge funds in native or ERC20 formats, and execute transactions with payments. Additionally, the UTB Module plays a crucial role in handling incoming bridge transactions from other chains.

Comprising three essential submodules, the UTB Module is organized as follows:

#### Core UTB
This module is tasked with user interaction, managing and processing user-initiated transactions, including swap and bridge operations.

#### Executer
The Executer module takes charge of facilitating payment transfers and executing arbitrary functions from other contracts.

#### Fee Collector
The Fee Collector module oversees the accumulation of platform fees associated with each swap and bridge operation. Additionally, it provides a mechanism for protocol owners to redeem the collected fees.

### Swapper Module

The Swapper module handles swapping operations, and it comprises two distinct submodules, namely:

#### UniswapV3
The Uniswap module is designed for executing multihop swaps specifically on Uniswap V3.

#### WETH Contract
This module is tasked with executing ETH-WETH transfers through the utilization of the WETH contract.

### Bridge Adapters

Bridge Adapters play a pivotal role in facilitating communication between LayerZero bridge modules, enabling inter-chain interactions. Serving as an interface between the Decent Bridge and the Decent Protocol, these adapters ensure seamless coordination. Two types of bridge adapters are employed for this purpose:

#### Decent Bridge Adapter
The Decent Bridge Adapter serves as the intermediary for communication with the Decent Bridge module, an implementation built on LayerZero.

#### Stargate Bridge Adapter
The Stargate Bridge Adapter is crafted to facilitate interaction with the Stargate Bridge, enabling seamless interchain communication.

### Decent Bridge

The Decent Bridge stands as the LayerZero implementation of a bridge, shouldering the responsibility for interchain communication.

## Recommendations

Presently, signature verification is integrated into the Fee Collector module. To enhance flexibility and accommodate potential future growth of the protocol, it is advisable to segregate these functionalities. This separation would render the Signer module more versatile, allowing for straightforward integration as the protocol expands and potentially incorporates additional signature verification processes.

## Auditing Approach

In the process of auditing the smart contracts, I scrutinized the provided contracts of decent protocol and documentations of decent protocol, stargate bridge protocol, stargate protocol and uniswapv3 protocol. My auditing approach involved a meticulous examination of the code base, conducting a comprehensive code review focusing on solidity and security. To ensure the contracts' functionality and resilience, I actively compiled the code and tested various scenarios. This detailed analysis enabled me to evaluate the contracts' performance and security measures in diverse scenarios. Additionally, I verified the main invariants, explored attack ideas presented by the protocol, and considered additional contextual information as part of my thorough audit approach.

## Codebase quality analysis

The codebase demonstrates a satisfactory level of quality, offering clarity in functionality through self-explanatory code and the use of straightforward logic. While the implementation of simple functionalities contributes to its comprehensibility, the absence of well-documented comments impacts overall readability. Notably, contracts like UTB.sol exhibit thorough documentation and comments, enhancing their clarity. However, certain components such as bridge adapters and Uniswappers lack comprehensive documentation, affecting their overall understandability for both auditors and developers.

The absence of essential documentation and inline comments in the code has resulted in an increased learning curve and auditing time.

### Modularity

The codebase showcases a notable degree of modularity by proficiently isolating intricate additional logics into distinct functions, notably within core components such as UTB contracts, swappers, bridge adapters, and the bridge itself. Furthermore, there is an opportunity to enhance modularity by considering the separation of the signer module from the fee collector contract. This approach would contribute to a more modular and maintainable structure within the codebase.

### Comments

Insufficient comments throughout the contracts limit the clarity of each function, requiring auditors and developers to invest more time in understanding the functionality. Improving comments to thoroughly explain the various functionalities would greatly enhance the code's accessibility and comprehension.

### Access controls

The implementation of access control in the codebase is robust, ensuring seamless and conflict-free execution of functions. However, a critical oversight has been identified in the setRouter() function within DcntEth.sol, where a vital access control check is absent. This vulnerability creates a potential risk, as any user could set the router in DcntEth without proper authorization. Exploiting this loophole, an attacker could potentially disrupt the platform persistently, posing a significant threat.

### Error handling

The contracts exhibit a moderate level of error handling, but there are instances where critical checks for error management are lacking. Specifically, within the `UTB` contract, the mappings `swappers` and `bridgeAdapters` associate numeric identifiers with contract addresses for swappers and bridge adapters. The contract employs these mappings to cast addresses to their respective interfaces and engage with them. However, there is a notable absence of checks to ensure that these addresses are non-zero before casting and interacting with them. Failing to validate these addresses could result in transactions failing and reverting when attempting to interact with the zero address as though it were a valid contract. Implementing appropriate checks is essential to prevent such errors.

### Modifiers

Certain issues have been identified concerning the best practices for modifiers within the codebase. Notably, some modifiers play crucial roles in executing critical functions and managing state changes. For instance, in the `DecentEthRouter` contract, the `userDepositing` and `userIsWithdrawing` modifiers are utilized to update the `balanceOf` mapping, which monitors the balance of each user's deposited funds. These modifiers are employed to modify the user's balance either before or after the execution of the underlying function logic.

### Interfaces

The current codebase utilizes interfaces, but there is room for improvement in terms of documentation. The existing interfaces lack comprehensive documentation, making it challenging to understand the code and its functionalities. Enhancing the documentation for these interfaces would contribute to a clearer understanding of the codebase, promoting better transparency and ease of comprehension for developers and other stakeholders.

### Libraries

Presently, the contracts do not make use of any libraries. Nevertheless, it is recommended

 to explore the incorporation of a dedicated library to encapsulate the signing functionality. This approach can contribute to improved modularity and maintainability within the codebase, providing a structured and reusable foundation for the signing-related operations.

## Centralisation Risk

This protocol presents a notable risk of a single point of failure, as a majority of the contracts utilize Solmate's Owned contract for implementing ownership. This approach introduces a vulnerability, where if the owner is compromised, the entire system becomes susceptible to compromise.

### Recommendations

To address this risk, it is recommended to implement OpenZeppelin's Ownable2Step contract. This step mitigates centralization concerns and enhances the security of the protocol.

## Systemic Risk

### Lack of Upgradability Provisions

The contract is not currently structured with upgradability in mind. Should vulnerabilities or the need for improvements be identified, deploying updates to the contract could be problematic. It is advisable to consider integrating upgradability features to facilitate future enhancements and fixes.

### Absence of Pausability Feature

The contract currently lacks a pausability mechanism, which is a critical feature for emergency stopping of contract functions in case of vulnerabilities or attacks. The inclusion of such a feature would allow for a swift response to protect user funds and maintain system integrity during unforeseen events.


### Time spent:
30 hours