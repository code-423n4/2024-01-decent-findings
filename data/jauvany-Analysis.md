**Overview**

Decent allows for single click transactions on any chain, paid for from any source chain / token. i.e. suppose I want to mint an NFT on optimism, but only have funds on Base, I can send that transaction, paying with DAI on Base, to receive my funds on Optimism.

**Scope**

> The engagement involved analysing eleven (11) different Solidity Smart Contracts:

- UTB.sol: Calls swapAndExeucte and bridgeAndExecute

- UTBExecutor.sol: Calls the executor for esxternal contract calls

- UTBFeeCollector.sol: Collects fees on UTB contract calls

- BaseAdapter.sol: Standard functionality for each bridge adapter

- DecentBridgeAdapter.sol: adapter impl for decent bridge




- StargateBridgeAdapter.sol: adapter impl for stargate bridge

- SwapParams.sol: params for swapper files

- UniSwapper.sol: implementation of ISwapper for UniV3

- DcntEth.sol: OFTV2 implementation for DcntEth 

- DecentEthRouter.sol: Core bridge logic 

- DecentBridgeExecutor.sol: makes external contract calls

# Codebase quality analysis
 
> Analysis of the codebase (What’s unique? What’s using existing patterns?):

**Unique:* 
- Codebase carries out specific governance mechanisms that are uniquely designed for Decent’s specific use case e.g  Decent allows for single click transactions on any chain, paid for from any source chain / token.

**Existing Patterns:* 
- The Decent Protocol adheres to common contract management patterns, such as the use of OnlyOwner, OnlyUtb  and OnlyExecutor.

**Strengths:*
- Named imports of parent contracts are well utilised.
- Rich documentation provided

**Weaknesses:*
- Incomplete test coverage (75%). 100% test coverage is recommended  
- All contracts in scope use floating solidity pragma versions, this is not recommended.
 
# Mechanism review 

The mechanisms implemented in the Decent ecosystem, including the ERC721 standard is innovative and well-designed. It provides a comprehensive range of features and capabilities. However, there are some potential issues and risks associated with these mechanisms. These issues should be carefully considered and mitigated to ensure the security and reliability of the ecosystem. 
 
# Centralization risks
 
Privileged functions can create points of failure: Ensure such accounts are protected and consider implementing multi-sig to prevent a single point of failure. See [Here](https://github.com/code-423n4/2024-01-decent/blob/main/bot-report.md) for complete details. 

# Systemic risks

- External Contract Dependencies: Decent relies on external contracts from OFTV2, Uniswap and Solmate. If any of these contracts have vulnerabilities, it would affect the protocol.

- Test Coverage: The test coverage provided by Decent is 75% however, 100% tests coverage is recommended.

- Like any smart contract-based system, Decent is exposed to potential coding bugs or vulnerabilities. Exploiting these issues could result in the loss of funds or manipulation of the protocol. 

# Documents 

The documentation of the Decent protocol is quite comprehensive and detailed, providing a solid overview of how the Decent protocol is structured and how its various aspects function. 

I would also recommend adding quality Medium articles, it’s a great way to provide an indepth look at many of the topics in the project and is used by many blockchain projects.

# Automated Findings 

See automated findings [here](https://github.com/code-423n4/2024-01-decent/blob/main/bot-report.md)
The 4naly3er report can be found [here](https://github.com/code-423n4/2024-01-decent/blob/main/4naly3er-report.md).

# Architecture recommendations

> The Decent architecture seems solid in general, none the less here are some areas that could be improved:

- Testing and Simulations: I recommend creating a live testnet app. Here is an
[example](https://app.opendollar.com/) from The Open Dollar protocol. Conduct thorough testing of all contracts and functions and simulations to understand how they will behave under various market conditions.

- I recommend rewriting some of the tests in the codebase for this audit to use the actual contracts instead of mock addresses like in some cases. This will offer greater confidence during system deployment.

**Gas Optimizations**

- Review data types: Analyze the data types used in your smart contracts and
consider if they can be further optimized. For example, changing uint256 to
uint128 or uint94 can save gas and storage slots.
- Struct packing: Look for opportunities to pack structs into fewer storage slots. By carefully selecting appropriate data types for struct members, you can reduce the overall storage usage.
- Use constant values: If certain values in your contracts are constant and do
not change, declare them as constants rather than storing them as state variables. This can significantly save gas costs.
- Avoid unnecessary storage: Examine your code and eliminate any unnecessary storage of variables or addresses that are not required for contract functionality.
- Storage vs. memory usage: When working with arrays or structs, consider whether using storage instead of memory can save gas. Using storage allows direct access to the state variables and avoids unnecessary copying of data.
- Replacing the use of memory with calldata for read-only arguments in external  functions.

**Other recommendations**

- Regular code reviews and adherence to best practices.
- Conduct external audits by security experts.
- Consider open sourcing the contract for community review.
- Maintain comprehensive security documentation.
- Establish a responsible disclosure policy for vulnerabilities.
- Implement continuous monitoring for unusual activity.
- Educate users about risks and best practices.

# Approach taken in evaluating the codebase
 
> My analysis of the Decent Protocol Included understanding the architecture, mechanism,overall codebase and possible risks associated to the protocol.

- Day 1: I spent time reading the different available documentation in order to have a deep understanding of the protocol. 

- Day 2: I analyzed the codebase for better  understanding, Performed a Mechanism review and investigated possible systemic risks, and centralization risks. 

- Day 3: I dedicated this day to coming up with possible Architecture recommendations after identifying possible risks and prepared the final analysis report.

**Conclusion**  
The Decent ecosystem is a well-designed and innovative platform for crypto transactions, offering a wide range of features and capabilities. However, there are areas where improvements could be made, particularly in terms of permission management, gas efficiency, and potential exploitation of certain modules. By addressing these issues, the Decent ecosystem could become even more secure, efficient, and reliable






### Time spent:
24 hours