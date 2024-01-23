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
If `amt2Bridge` is greater than `swapParams.amountIn`, then the approval given is more than what is actually needed for the operation. This scenario is typically referred to as "over-approval". Note: This will currently not affect anything as unlike [StargateBridgeAdapter.getBridgedAmount](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L61-L67) that returns a reduced [newPostSwapParams.amountIn](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L186-L188), `DecentBridgeAdapter.getBridgedAmount()` returns `amountOut` which is `amount2Bridge` by default:

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L73-L79

```solidity
    function getBridgedAmount(
        uint256 amt2Bridge,
        address /*tokenIn*/,
        address /*tokenOut*/
    ) external pure returns (uint256) {
        return amt2Bridge;
    }  
```
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

## [NC-01] Inefficient token handling in UTB.sol
In UTB.sol, the `performSwap` function is private and only called internally within the contract, and considering that all internal calls to this function have the [retrieveTokenIn](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L65) parameter hardcoded to true, it does seem unnecessary to have this as a parameter.

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L55

```solidity
        return performSwap(swapInstructions, true);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L82

Specifically, the following `else if` may simply be replaced with `else` for cleaner code:

```solidity
        } else if (retrieveTokenIn) {
```
## [NC-02] Ensure Amount Redeemed is Available
In the UTBFeeCollector smart contract, a suggested enhancement is the incorporation of balance checks within the `redeemFees` function. This modification ensures that the contract confirms the sufficiency of its holdings — whether in native Ether or ERC20 tokens — before proceeding with any transfer operations. Specifically, for Ether, it verifies the contract's Ether balance against the requested amount, and for ERC20 tokens, it checks the contract's token balance. By integrating these checks, the smart contract prevents the execution of transactions that would inevitably fail due to inadequate funds. 

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L69-L75

```solidity
    function redeemFees(address token, uint amount) public onlyOwner {
        if (token == address(0)) {
            payable(owner).transfer(amount);
        } else {
            IERC20(token).transfer(owner, amount);
        }
    }
```
## [NC-03] Activate the Optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.0",
settings: {
optimizer: {
  enabled: true,
  runs: 1000,
},
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.
