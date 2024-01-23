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

## [L-03] Enhancing Security and Reliability in Low-level Ether Transfers
The use of low-level `call()` for transferring Ether presents risks, particularly when dealing with contracts lacking `receive()` or those intentionally designed to fail transactions via gas griefing leading to unnecessary DoS. To mitigate these risks, implementing a gas limit on `call()` could prevent gas griefing, while transitioning to Wrapped Ether (`weth`) offers a more robust and ERC20-compliant transfer method, avoiding issues with contracts not equipped to handle direct Ether transfers.

For example, the following instance may be refactored as follows:

https://github.com/code-423n4/2024-01-decent/blob/main//src/UTBExecutor.sol#L54-L54

```diff
-                (refund.call{value: amount}(""));
+                assembly {
+                    // Transfer ETH to the recipient
+                    // Limit the call to 50,000 gas
+                    success := call(50000, refund, amount, 0, 0, 0, 0)
+                }

+                // If the transfer failed:
+                if (!success) {
+                    // Wrap as WETH
+                    weth.deposit{ value: amount }();

+                    // Transfer WETH instead
+                    weth.transfer(refund, amount);
+                }
```  
## [L-04] Over-Approval Risks in DecentBridgeAdapter.bridge()
`DecentBridgeAdapter.bridge()` approves the router to spend `amt2Bridge` amount of `bridgeToken` on behalf of the `DecentBridgeAdapter` contract when `swapParams.amountIn` is the actual amount involved:

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L114-L124

```solidity
            IERC20(bridgeToken).approve(address(router), amt2Bridge);
        }

        router.bridgeWithPayload{value: msg.value}(
            lzIdLookup[dstChainId],
            destinationBridgeAdapter[dstChainId],
            swapParams.amountIn,
            false,
            dstGas,
            bridgePayload
        );
```
In `DecentEthRouter._bridgeWithPayload()`, the amount that is being transferred is `swapParams.amountIn` (where _amount is equivalent to swapParams.amountIn in the context of the bridge operation):

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L181

```solidity
            weth.transferFrom(msg.sender, address(this), _amount);
```
If `amt2Bridge` is greater than `swapParams.amountIn`, then the approval given is more than what is actually needed for the operation. This scenario is typically referred to as "over-approval".

Over-approval in itself is not a direct security vulnerability, but it can lead to potential risks. If the router or any entity with control over the router becomes malicious or is compromised, they could potentially use this excess approval to transfer more tokens than necessary for the intended operation, up to the approved amount (`amt2Bridge`).

As a best practice in smart contract development, it's generally recommended to approve only the necessary amount for a transaction. This minimizes risks and follows the principle of least privilege. In this case, adjusting the approval to `swapParams.amountIn` would be more aligned with this best practice, assuming `swapParams.amountIn` is always less than or equal to `amt2Bridge`.

## [L-05] Best Practices on Refund Mechanism
In Solidity, implementing a robust and secure refund mechanism is crucial, particularly when interacting with arbitrary external addresses. The practice of not checking the return value of a low-level call poses significant risks as evidenced in the following instances:

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol

```solidity
54:                (refund.call{value: amount}(""));

67:                (refund.call{value: extraNative}(""));
```
Much as the function NatSpec indicates that `refund` is typically the EOA that initiated the transaction, `refund` could also be a contract. This approach can lead to failed refunds without notice if the recipient is a contract with complex behaviors or insufficient gas. Moreover, ignoring the return status of such calls can exacerbate vulnerabilities like reentrancy attacks. Hence, it's advised to always check the return value of `.call`, potentially using `require` or `if` to revert the transaction on failure, or alternatively, log the failure with an event. Implementing these best practices ensures greater reliability and security in handling refunds within smart contracts.

## [L-06] The Impacts of Omitting Deadline in Uniswap Transactions
In Uniswap transactions, the deadline parameter serves as a critical safeguard, setting a specific timestamp until which a transaction remains valid. Omitting this parameter, as shown in the instance below, carries significant implications. 

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L153

```solidity
                //deadline: block.timestamp,

```
Primarily, it increases susceptibility to front-running attacks, where malicious actors capitalize on seeing pending transactions and act in ways that unfavorably alter market conditions before the transaction is executed. Additionally, without a deadline, a transaction may execute under drastically different market conditions than anticipated, due to the inherent volatility of cryptocurrency markets. This removal of temporal constraints means the transaction can theoretically be executed at any point in the future, introducing uncertainty of unfavorable prices and potential financial risks. Therefore, while there might be niche scenarios where excluding a deadline is intentional, its omission generally represents a substantial deviation from standard safe trading practices in decentralized finance.