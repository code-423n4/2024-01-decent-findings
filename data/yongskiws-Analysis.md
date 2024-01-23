
# Decent Audit Analysis
![download (1)](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/87606034-dcf1-488e-8436-81b09ad03738)

Decent provides a multi-chain NFT infrastructure for easy creation and transactions. Developers can focus on their applications with well-documented SDKs, abstracting Solidity and enabling cross-chain transactions. Creators use the code-free Creator HQ to deploy, manage, and analyze NFT collections. Collectors explore Decent collections on Ethereum, Arbitrum, Optimism, and Polygon through our API. The Box facilitates 1-click checkout for cross-chain NFT purchases, meeting collectors at their liquidity point. It can be implemented with a single React component.


# Overview

Structuring the code is crucial for conducting a thorough analysis of the code base. This enables auditors to predict the level of contract difficulty, identify critical contracts, and determine security-critical features such as payment functions or assembly usage. Additionally, this approach accurately measures the time required to perform a detailed audit and helps determine the cost of a project audit. To achieve efficiency, clarity, and improvements, it is essential to have a comprehensive view of the entire architecture. This enables the identification of the basic structure, relationships between modules, and key design patterns. By understanding the big picture, we can analyze the complexity of the code and reveal its strengths and potential for improvement with confidence.

Lines: total lines of the source unit
nLines: normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
nSLOC: normalized source lines of code (only source-code lines; no comments, no blank lines)
Comment Lines: lines containing single or block comments
Complexity Score: a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

| Type | File                                      | Lines | nLines | nSLOC | Comment Lines | Complex. Score | Capabilities |
|------|-------------------------------------------|-------|--------|-------|--------------|----------------|--------------|
| 📝   | UTBFeeCollector.sol                       | 80    | 74     | 44    | 20           | 68             | 🖥💰📤🧮🔖         |
| 📝   | UTBExecutor.sol                           | 82    | 67     | 37    | 20           | 38             | 💰📤           |
| 📝   | UTB.sol                                   | 342   | 293    | 183   | 79           | 118            | 💰            |
| 📝   | swappers\UniSwapper.sol                   | 174   | 148    | 119   | 4            | 94             | 💰📤           |
| 📚   | swappers\SwapParams.sol                   | 20    | 20     | 13    | 5            | 3              |              |
| 📝   | decent-bridge\DecentEthRouter.sol         | 340   | 271    | 221   | 23           | 146            | 💰📤           |
| 📝   | decent-bridge\DecentBridgeExecutor.sol    | 89    | 73     | 41    | 20           | 42             | 💰📤           |
| 📝   | decent-bridge\DcntEth.sol                 | 39    | 39     | 27    | 4            | 24             |              |
| 📝   | bridge_adapters\StargateBridgeAdapter.sol | 221   | 178    | 147   | 18           | 82             | 💰            |
| 📝   | bridge_adapters\DecentBridgeAdapter.sol   | 159   | 128    | 106   | 4            | 73             | 💰            |
| 📝   | bridge_adapters\BaseAdapter.sol           | 22    | 22     | 16    | 1            | 10             |              |
| 📝📚  | Totals                                    | 1568  | 1313   | 954   | 198          | 698            | 🖥💰📤🧮🔖         |



### Diagrams Risk
![diagram risk decents](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/c71d9d22-8475-4702-849d-8b5dda14c7ac)

### Diagrams Source Lines (sloc vs. nsloc)
![Source Lines Editor](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/06caa78f-f6be-4e5e-898e-98a17be5c907)

### Exposed Functions

| 🌐Public | 💰Payable |
|----------|-----------|
| 70       | 27        |

| External | Internal | Private | Pure | View |
|----------|----------|---------|------|------|
| 22       | 57       | 20      | 12   | 6    |

### StateVariables

| Total | 🌐Public |
|-------|----------|
| 39    | 25       |

### Capabilities

| Solidity Versions observed | 🧪 Experimental Features | 💰 Can Receive Funds | 🖥 Uses Assembly         | 💣 Has Destroyable Contracts |
|----------------------------|--------------------------|-----------------------|--------------------------|-----------------------------|
| ^0.8.0, ^0.8.13            |                          | yes                   | yes, (1 asm blocks)     | -                           |


| 📤 Transfers ETH | ⚡ Low-Level Calls | 👥 DelegateCall | 🧮 Uses Hash Functions | 🔖 ECRecover | 🌀 New/Create/Create2 |
|------------------|---------------------|-----------------|------------------------|-------------|-----------------------|
| yes              | -                   | -               | yes                    | yes         | -                     |


| ♻️ TryCatch | Σ Unchecked |
|------------|-------------|
| -          | -           |


### Dependencies / External Imports


| Dependency / Import Path                                        | Count |
|-----------------------------------------------------------------|-------|
| @uniswap/swap-contracts/interfaces/IV3SwapRouter.sol             | 1     |
| decent-bridge/src/interfaces/IDecentEthRouter.sol                | 1     |
| decent-bridge/src/interfaces/IWETH.sol                          | 2     |
| forge-std/interfaces/IERC20.sol                                  | 6     |
| solidity-examples/token/oft/v2/OFTV2.sol                         | 1     |
| solidity-examples/token/oft/v2/interfaces/ICommonOFT.sol         | 1     |
| solidity-examples/token/oft/v2/interfaces/IOFTReceiverV2.sol     | 1     |
| solmate/auth/Owned.sol                                           | 5     |


### AST Node Statistics
![function calls](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/5d9ae189-9146-4d9d-9a21-9742b2e8a149)
![AST ELEMENTS](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/b2e5a788-d96b-43c7-bd9e-7b3c5b8be3ad)
![Assembly](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/b3fadf4e-31d6-4355-b891-fc1c3982595e)




# Approach to Auditing
  - We began by thoroughly reviewing and read docs decent procotol the provided documentations, as well as reviewing the video provided. Here, we made note of important information, noted down unclear parts and overall tried to fully understand how the protocol should function. We also took notes of integrated and inherited protocols and mapped out possible risk areas.

  - While we were studying the docs, we had ran static analyzers and linters through the contracts to detect basic vulnerabilities and other non critical errors. The result of our static analysis was then compared to the provided bot reports and those deemed known were removed. 

  - After the documentation review and static analysis, we started the process of manual code inspection. We noted how the contracts were divided into different modules and we inspected the contracts in each module carefully. We stuidied each of the contracts, tested how each function behaves and compared that to how they're expected to behave. We then tried out various attack ideas, including the ones provided by the sponsors. We noted the dependencies, inheritancies, and possible vulnerabilities that can arise from them. We made comparisons to issues found in protocols of the same kind (including older commits) to see if the same mistakes were repeated and how effective the provided fixes were. 

  - After this was done, we made notes on the issues we found and prepared the analysis report.



# Codebase quality
Overall, I consider the quality of the Decent codebase to be excellent. The code appears to be very mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The codebase demonstrates moderate maintainability with well-structured functions and comments, promoting readability. It exhibits reliability through defensive programming practices, parameter validation, and handling external calls safely. The use of internal functions for related operations enhances code modularity, reducing duplication. Libraries improve reliability by minimizing arithmetic errors. Adherence to standard conventions and practices contributes to overall code quality. However, full reliability depends on external contract implementations like openzeppelin, uniswap.                                                                       |
| **Code Comments**                        | The contracts have comments that are used to explain the purpose and functionality of different parts of the contracts, making it easier for developers to understand and maintain the code. The comments provide descriptions of methods, variables, and the overall structure of the contract.For-Exmaple: The code comments in the "UTB" contract provide essential information. The imported interfaces and contracts are declared, and the contract's title and purpose are described.                                                                                                                                                                         |
| **Testing**                              | The audit scope of the contracts to be audited is 75% and it should be aimed to be 100%.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |

### Strengths :
Exemplary unit test coverage: The codebase shines with its meticulous unit testing strategy. The comprehensive suite of unit tests ensures the reliability and resilience of the system's core functionality, providing confidence in its performance under various scenarios.

Artful integration of Natspec: A commendable strength is the thoughtful integration of Natspec. The use of this documentation format not only improves code readability, but also exemplifies a commitment to providing clear and human-friendly documentation that fosters a more collaborative and accessible development environment.

### Weaknesses :
Opportunities to improve documentation: While the codebase stands out in many ways, there is an opportunity for refinement in the documentation. Addressing this area of improvement can increase the overall clarity of the codebase, make it more accessible, and facilitate seamless collaboration among developers. In essence, the codebase demonstrates proficiency in testing methodologies and documentation practices. Strengthening the documentation further would strengthen the codebase's comprehensibility and contribute to an even more polished and developer-friendly ecosystem.


# Approach we followed when reviewing the code

`UTB` designed to facilitate cross-chain token exchange with several functionalities such as token exchange, transaction execution with payment, and token bridge creation.

`UTBExecutor` aims to simplify the process of executing payment transactions, both with native and ERC20 tokens, by paying attention to security aspects and the authority given to the owner

`UTBFeeCollector` to collect and handle fees in native or ERC20 form using signatures for verification. This contract gives the owner the ability to redeem (redeem) the fees that have been collected

`UniSwapper` designed to perform token swaps using the Uniswap protocol with several different exchange scenarios. This contract gives the owner the ability to manage the Uniswap router address and wrapped token

`DecentBridgeAdapter` to interact with the DecentEthRouter router to perform on-chain asset exchanges and facilitate the transfer of fees in transactions. This contract has several functions focused on integration with specific bridges and routers.

`StargateBridgeAdapter` designed to interface with the StargateRouter router to perform on-chain asset exchange using the Stargate protocol. This contract has several functions that focus on integration with certain Stargate bridges and routers

`DecentEthRouter` manage on-chain bridge processes using DCNT and Ether/WETH tokens. These contracts facilitate the transfer of assets and liquidity between different chains


# Analysis of the code base and diagram architecture

## UTB
![UTB SOL](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/b99f4b91-49db-4cae-b92d-05b9d0a396f9)

## UTBExecutor
![UTBExecutor](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/513f779a-d6bc-4491-bbc7-b6eaa660ffa5)

## UTBFeeCollector
![UTBFeeCollector](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/df25ab1e-fda9-4316-8d3d-40a539ae7a8f)

## UniSwapper
![UniSwapper](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/1e9c7e41-042e-40a5-9ee9-61a7d5650cee)

## DecentBridgeAdapter
![DecentBridgeAdapter](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/635b7aff-1c37-431b-9cba-e8aa84e1ac97)

## StargateBridgeAdapter
![StargateBridgeAdapter](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/201b7fec-2188-440e-90ee-dacab53f4963)

## DecentEthRouter
![DecentEthRouter](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/ebf97659-bc36-40c0-8dce-cbf2feeb8dcc)

# Risk Assessment:

## Centralized Point of Failure
`DecentBridgeExecutor`for executing cross-chain transactions. If the `DecentBridgeExecutor` contract is compromised or experiences downtime, it could impact the functionality of the `DecentEthRouter` contract.

## Gas Currency Risks
supports different gas currencies, which could introduce additional risks related to price volatility, liquidity, and slippage. Ensuring adequate liquidity and monitoring gas currency prices is essential to maintain the contract's functionality.

## Solidity Version
The contract interacts with various external contracts, which could contain bugs or vulnerabilities. Regularly updating the contract and its dependencies can help minimize this risk.

## Dependency Risks
external libraries and interfaces, which could introduce additional risks related to version compatibility, security, and maintenance. Ensuring that the dependencies are up-to-date, well-maintained, and secure is crucial for the contract's long-term viability.


# Conclusion
In general, the Decent project exhibits an interesting and well-developed architecture we believe the team has done a good job regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from potential malicious use cases. Additionally, it is recommended to improve the documentation and comments in the code to enhance understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.

### Time spent:
22 hours