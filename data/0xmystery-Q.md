## [L-01] The Imperative of Correct payable Visibility Usage
The precise use of the `payable` visibility is crucial for both security and user clarity. A notable example is the `addLiquidityWeth` function in DecentEthRouter.sol, often erroneously marked as payable despite being designed exclusively for ERC-20 token transactions and not for handling native Ethereum (ETH). This misalignment can lead to user confusion and unintentional ETH transfers, posing a risk of lost funds. While it might at times be deliberately included in functions accessible only to restricted parties such as the owner for minor gas efficiency reason, removing the payable modifier in such scenarios is not only a best practice but also an essential step in enhancing contract security. It ensures that the function's capabilities are strictly aligned with its intended purpose, thereby safeguarding user assets and maintaining the integrity of the contract's operation.

Here's a specific instance entailed:

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L322-L327

```solidity
    function addLiquidityWeth(
        uint256 amount
    ) public payable userDepositing(amount) {
        weth.transferFrom(msg.sender, address(this), amount);
        dcntEth.mint(address(this), amount);
    }
```
## [L-02] Cross-Chain Deployment Challenges Due to Solidity Version ^0.8.xx
The protocol's contracts, compiled with floating Solidity version ^0.8.xx, may face deployment and operational challenges on blockchain networks that are not compatible with the Ethereum Shanghai hard fork. This compatibility issue stems from the use of new opcodes introduced in the hard fork, specifically integrated into Solidity 0.8.20 and later versions.

According to the following link:

https://github.com/ethereum/solidity/releases/tag/v0.8.20

"Solidity 0.8.20 incorporates changes aligned with the Ethereum Shanghai hard fork, including the introduction of new EVM opcodes like `PUSH0`. Contracts compiled with this version of Solidity might produce bytecode that is not compatible with L2 blockchain networks, ie. Polygon and Avalanche the protocol intends to begin with, that have not implemented these changes. This incompatibility arises because such networks do not recognize or properly execute these new opcodes."

The severity is significant for projects aiming for multi-chain deployment and operation, particularly on chains that are not updated with the latest Ethereum enhancements. The primary concern is the inability to deploy or correctly operate these contracts on non-Shanghai-compatible chains, limiting the reach and functionality of the project. This could also become a problem if different versions of Solidity are used to compile contracts for different chains. The differences in bytecode between versions can impact the deterministic nature of contract addresses, potentially breaking counter-factuality.

To ensure broader compatibility across various blockchain networks, consider downgrading the Solidity compiler version to 0.8.19 or earlier, which does not include Shanghai hard fork specific opcodes. This ensures compatibility with a wider range of blockchain networks.

Here're some of the instances found:

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L2

```solidity
pragma solidity ^0.8.0;
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L2

```solidity
pragma solidity ^0.8.13;
```
Note: Hardhat has introduced a feature or update to circumvent issues arising from the Shanghai hard fork (referred to as "Paris over Shanghai") which reflects an ongoing effort to maintain compatibility and ease of development across different Ethereum versions and forks. Despite these tools, it’s still crucial for developers to be mindful of the Solidity compiler version they use. Even with Hardhat's solutions, there may be scenarios where using a specific compiler version is necessary to ensure compatibility, particularly when dealing with multiple blockchains with differing levels of support for Ethereum's latest features. 

## [L-03] Addressing transfer and send Limitations in zkSync
According to "Additional Context" in the contest readme: 

"Will be deployed to most blockchains, can consider scope of blockchains to those supported by layerzero for now"

While the use of `call` vs `transfer` will be considered as a protocol choice, using `transfer()` will ready revert on zkSync Era whether or not the `fallback()` or `receive()` function of the receiving contract implements overly complex logic limited by the 2300 gas stipend. Please visit the following link for further insight:

https://medium.com/coinmonks/gemstoneido-contract-stuck-with-921-eth-an-analysis-of-why-transfer-does-not-work-on-zksync-era-d5a01807227d

"... the problem lies in the fact that zkSync Era is not fully compatible with the Ethereum Virtual Machine (EVM). Gas calculations on zkSync Era use a dynamic and divergent gas measurement method. Using transfer() on zkSync Era will exceed the 2300 gas limit, causing the transaction to revert automatically."


    
