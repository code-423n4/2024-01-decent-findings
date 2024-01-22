# 🛠️ Analysis - Decent Audit 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |Other recommendations | What is unique? How are the existing patterns used? |
|h) |New insights and learning from this audit | Things learned from the project |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2024-01-decent

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2024-01-decent?tab=readme-ov-file#tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Decent](https://docs.decent.xyz/) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| The project does not currently have a slither result, a slither control was created from initial|
|5|Test Suits|[Tests](https://github.com/code-423n4/2024-01-decent?tab=readme-ov-file#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2024-01-decent?tab=readme-ov-file#scope)|
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2024-01-decent?tab=readme-ov-file#attack-ideas-where-to-look-for-bugs)||

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit

In addition, the auditor can decide on the question of where should I end the audit by examining these metrics;

Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

<img width="734" alt="image" src="https://github.com/code-423n4/2024-01-decent/assets/104318932/f83040ff-3716-45d1-9276-0dc9afe6f570">







</br>
</br>

## Project - Entry Point;

- Core Bridge structure above; It is used for token bridging from one blockchain to another in the Decent project.

- The user initiates the bridging process by calling the `bridgeWithPayload` or `bridge` functions.

- While `bridgeWithPayload` is used for more complex scenarios by carrying additional data (payload), `bridge` is preferred for simpler transactions.

- Both functions call the internal `_bridgeWithPayload` function.

- This function creates bridging parameters via `_getCallParams` and performs the bridging process by passing these parameters to the `dcntEth.sendAndCall` function.

- This process ensures the secure transfer of tokens to the target chain and manages the details of the bridging process (target address, quantity, gas cost).

##### Note: Please click on the image to enlarge it:
<img width="1581" alt="image" src="https://github.com/code-423n4/2024-01-decent/assets/104318932/897210e8-0b08-4253-9582-7d5eede617ba">


</br>
</br>
</br>
</br>

## DecentEthRouter.sol  (except bridge);

- redeemEth: The user sends a certain amount of ETH to the smart contract. The contract converts this amount of ETH into WETH and sends it back to the user.

- redeemWeth: The user sends a certain amount of WETH to the smart contract. The contract transfers this amount of WETH to the user's account.

- addLiquidityEth: User provides liquidity by depositing ETH. The contract converts the deposited ETH amount into WETH and issues dcntEth tokens in return.

- removeLiquidityEth: The user withdraws a certain amount of liquidity (dcntEth tokens). The contract burns these tokens and in return sends ETH back to the user.

- addLiquidityWeth: User provides liquidity by depositing WETH. The contract receives this amount of WETH and issues dcntEth tokens in return.

- removeLiquidityWeth: The user withdraws a certain amount of liquidity (dcntEth tokens). The contract burns these tokens and in return sends WETH back to the user.

- This structure allows users to provide and manage liquidity with Ethereum and WETH


##### Note: Please click on the image to enlarge it:
<img width="1830" alt="image" src="https://github.com/code-423n4/2024-01-decent/assets/104318932/1db42b41-e5fe-48fa-a319-295f1c0ea94c">


</br>
</br>
</br>
</br>


## DecentBridgeExecutor.sol ;

The transaction begins with the transfer of some tokens to the `DecentBridgeExecutor` contract (`Executor`). Then, two different paths can be followed depending on whether ETH is used as gas currency and whether ETH delivery is made:

1. **`_executeWeth` Path**: If `gasCurrencyIsEth` is `false` or `deliverEth` is `false`, `_executeWeth` function is called. This function validates WETH tokens to the target contract (`Target`) and then makes a call to the target contract. If the transaction is successful, the remaining amount of WETH is refunded to the user.

2. **`_executeEth` Path**: If `gasCurrencyIsEth` and `deliverEth` are both `true`, `_executeEth` function is called. This function withdraws WETH and then makes a call to the target contract worth ETH. If the transaction fails, the ETH amount is returned to the user.


##### Note: Please click on the image to enlarge it:
<img width="1706" alt="image" src="https://github.com/code-423n4/2024-01-decent/assets/104318932/20c202a7-818f-4b2c-a5a4-f554f3439735">




## c) Test analysis
### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content.

Structure and Approach of Tests
Multi-contract Integration: Tests use multi-contract integration by inheriting from other contracts (e.g. `UTB`, `XChainExactInFixture`). This better simulates real-world usage scenarios and allows testing contract interactions.

Environment Settings: Test files have the ability to switch between different blockchain environments (such as optimism, arbitrum, polygon, etc.). This is used to test the behavior of the contract on different chains.

Detailed Scenario Tests: Contracts test various transaction scenarios (e.g. `testExactInEth2Weth`, `testExactInUsdc2Eth`). This is critical to understanding how the contract behaves under different inputs and conditions.

Gas Usage and Optimization: Performance metrics such as gas usage and optimization seem to be taken into account in the tests (variables such as `DEPOSIT_GAS` are used in this context).

Performance metrics such as gas usage and optimization have an important place in the tests. Especially in cross-chain transactions, gas usage is a critical factor in terms of cost and performance.




### What could they have done better?





-  1) If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;


<img width="784" alt="image" src="https://github.com/code-423n4/2024-01-decent/assets/104318932/868e9da2-a4b7-4431-bd13-5fa58474c488">


Ref: https://xin-xia.github.io/publication/icse194.pdf

</br>
</br>

-   2) Overall line coverage percentage provided by your tests : 75 (As a generally accepted safety rule, 100% test coverage is recommended)


</br>
</br>



-  3) Test Tools/Technologies analysis of the project;

<img width="985" alt="image" src="https://github.com/code-423n4/2024-01-decent/assets/104318932/218730f2-2dc9-40cc-9155-be50d0cc8cf1">

</br>
</br>

- 4) There is a deficiency in the test files regarding the testing of slippage;

For instance i can example with this function ; `UniSwapperTest.t.sol.testSwapUsdcToWETHExactIn()`

```solidity
test/swappers/UniSwapper.t.sol:
  57  
  58:     function testSwapUsdcToWETHExactIn() public {
  59:         // buys eth with 4 usdc
  60:         uint usdcAmount = 4e6;
  61:         mintUsdcTo(chain, alice, usdcAmount);
  62:         uint aliceBalance = usdcBalance(chain, alice);
  63: 
  64:         assertEq(aliceBalance, usdcAmount);
  65: 
  66:         UniSwapper swapper = deploySwapper();
  67: 
  68:         address usdc = getUsdc(chain);
  69:         address weth = getWeth(chain);
  70: 
  71:         (SwapParams memory swapParams, uint expected) = getSwapParamsExactIn(
  72:             chain,
  73:             usdc,
  74:             usdcAmount,
  75:             weth,
  76:             slippage
  77:         );
  78: 
  79:         startImpersonating(alice);
  80:         ERC20(usdc).approve(address(swapper), usdcAmount);
  81:         swapper.swapExactIn(swapParams, bob);
  82:         stopImpersonating();
  83: 
  84:         assertEq(wethBalance(chain, bob), expected);
  85:     }
```

In the testSwapUsdcToWETHExactIn() function of the UniSwapperTest contract, the uint slippage = 2; line sets a slippage value


Setting Slippage Tolerance: The slippage variable is set to 2, which likely represents a percentage. This means the transaction will tolerate a price movement of up to 2% from the quoted price at the time of transaction initiation.

Calculating Expected Outcome: The `getSwapParamsExactIn` function calculates the expected outcome of the swap, taking into account the slippage tolerance. This function returns parameters for the swap, including the minimum amount of WETH that Alice expects to receive for her USDC, considering the slippage.

`assertEq(wethBalance(chain, bob), expected);`
This line asserts that the amount of WETH received by `Bob` matches the expected amount calculated earlier. However, this assertion alone does not explicitly check if slippage occurred within the tolerated range. It only checks the final balance against the expected outcome.

To explicitly check for slippage, the contract would need additional logic to compare the actual execution price against the expected price and ensure it's within the 2% tolerance. This could involve calculating the actual price per unit of WETH received and comparing it to the expected price per unit, factoring in the slippage.

In summary, while the function sets a slippage tolerance and calculates expected outcomes based on it, it does not explicitly verify if the actual slippage was within the set tolerance. The assertion at the end only verifies the final balance against the expected amount, not the price at which the swap was executed. This situation is given as an example from a function and is valid for all slippage tests in the project.


</br>
</br>
</br>
</br>

## d) Security Approach of the Project

### Successful current security understanding of the project;

1- They manage the 1nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes. 
(The old contracts were audited by the "Arbitrary Execution" company in June 2023, but the audit subject to this newly changed audit has not been carried out yet.)


### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. 
https://immunefi.com/

5- Pause Mechanism
This is a chaotic situation, which can be thought of as a choice between decentralization and security. Having a pause mechanism makes sense in order not to damage user funds in case of a possible problem in the project.

6- Upgradability
There are use cases of the Upgradable pattern in defi projects using mathematical models, but it is a design and security option.

7- Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

8- ChainAnalysis oracle
With the ChainAnalysis oracle, OFAC interaction can be blocked so that legal issues do not arise

9- We also recommend that you have an "Economic Audit" for projects based on such complex mathematics and economic models. An example Economic Audit is provided in the link below;
Economic Audit with [Three Sigma](https://panoptic.xyz/blog/panoptic-three-sigma-partnership)

10 - As the project team, you can consider applying the multi-stage audit model.

<img width="498" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/7ad49e46-4998-4bf2-830e-711039ea97a8">

Read more about the MPA model;
https://mpa.solodit.xyz/

11 - I recommend having a masterplan applied to project team members (This information is not included in the documents).
All authorizations, including NPM passwords and authorizations, should be reserved only for current employees. I also recommend that a definitive security constitution project be found for employees to protect these passwords with rules such as 2FA. The LEDGER hack, which has made a big impact recently, is the best example in this regard;

https://twitter.com/Ledger/status/1735326240658100414?t=UAuzoir9uliXplerqP-Ing&s=19


12 -Another security approach I can recommend is 0xDjango's security approaches in a project. This approach divides security into layers (Test, Automatic Tool, Individual Audit, Corporate Audit, Economic Audit) and aims to have each part done separately by experts in the field. I can also recommend this approach to you;

https://x.com/0xDjangoOnChain/status/1748402771684909094?s=20

</br>
</br>

## e) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2024-01-decent/blob/main/bot-report.md

**Audit from Arbitrary Execution**
https://github.com/decentxyz/Box-Audit/blob/074260ee9bb71254248eef19482909937fd95c29/The_Box_Audit_July_2023.pdf

A summary of the key points from the Audit from 2023 document:

The critical issue found allows an attacker to steal user ERC20 tokens via a specially crafted transaction. This is due to an arbitrary call in the Dispatcher contract.

Other issues found included low severity issues like funds accumulating in the Core contract and incorrect documentation. Some notes on improvements to documentation and code quality were also made.

All identified issues appear to have been addressed by the Decent team based on the provided pull request summaries.

In summary, the audit assessed The Box smart contracts for security and best practices, identifying issues which were then fixed by the Decent team to improve the overall security of the platform.


</br>
</br>
</br>
</br>

## Centralization Risk

#### Owner Role:

Centrality Risk High: The Owner role is highly centralized since it is singular and possesses significant administrative powers. 

Single Point of Failure: If the Owner's credentials are compromised, the entire system could be at risk. Similarly, if the Owner acts maliciously or negligently, it could have a substantial negative impact on the protocol.



</br>
</br>
</br>
</br>


##  f) Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
[`solmate`](https://www.npmjs.com/package/solmate)                                                                                        | [![npm](https://img.shields.io/npm/v/solmate.svg)](https://www.npmjs.com/package/solmate)   | Owned.sol                                                                                 |  - The latest updated version is used. |


</br>
</br>



</br>
</br>




## g) Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

✅ All staff accounts for the project should have control policies that require 2FA and must use 2FA wherever possible.
100% comprehensive security cannot be achieved based on smart contract codes alone.
Implement a more comprehensive policy to enforce non-SMS 2FA.
You can find the latest Simswap attack on Code4rena and details about it in this article: 
https://medium.com/code4rena/code4rena-twitter-x-incident-8b7f308a555d


✅ I recommend you to set up a `system.sol` basic architecture where all contracts are registered.
The entire system can revolve around a single contract, like SystemRegistry. This is the contract that ties all the other contracts together, and from this contract we should be able to list all the other contracts in the system. It's almost like a registry. 



✅ Analyze the gas usage of each function under various conditions in tests to ensure efficiency and identify potential optimizations.	





✅  I recommend that you classify functions such as Public, External, Internal as follows, this is the most effective method to classify functions ; 
<img width="467" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/f6661c98-7595-4ae2-bc5b-b086dc4f3018">

https://x.com/PaulRBerg/status/1657098652987318292?s=20

</br>
</br>


✅ This listing makes imports more understandable;


```diff
DecentEthRouter.sol

- import {IWETH} from "./interfaces/IWETH.sol";
- import {IDcntEth} from "./interfaces/IDcntEth.sol";
- iimport {ICommonOFT} from "solidity-examples/token/oft/v2/interfaces/ICommonOFT.sol";
- iimport {IOFTReceiverV2} from "solidity-examples/token/oft/v2/interfaces/IOFTReceiverV2.sol";
- iimport {Owned} from "solmate/auth/Owned.sol";
- iimport {IDecentBridgeExecutor} from "./interfaces/IDecentBridgeExecutor.sol";
- iimport {IDecentEthRouter} from "./interfaces/IDecentEthRouter.sol";


+ // Interfaces
+ import {IWETH} from "./interfaces/IWETH.sol";
+ import {IDcntEth} from "./interfaces/IDcntEth.sol";
+ import {ICommonOFT} from "solidity-examples/token/oft/v2/interfaces/ICommonOFT.sol";
+ import {IOFTReceiverV2} from "solidity-examples/token/oft/v2/interfaces/IOFTReceiverV2.sol";
+ import {IDecentBridgeExecutor} from "./interfaces/IDecentBridgeExecutor.sol";
+ import {IDecentEthRouter} from "./interfaces/IDecentEthRouter.sol";

+ // Authorization
+ import {Owned} from "solmate/auth/Owned.sol";

```
</br>
</br>

✅ In a network, for example, in this example the Polygon network has been chosen, native token can be sent on its own, this is done with the `swapAndExecute` function, it is not possible to send to another address, they can only be sent to themselves, this can be prevented with a require or it can be prevented from doing this on the website, because nobody It does not send native tokens as its own account receiver and its own account sender, but it may happen by mistake

```solidity
src/UTB.sol:
  107       */
  108:     function swapAndExecute(
  109:         SwapAndExecuteInstructions calldata instructions,
  110:         FeeStructure calldata fees,
  111:         bytes calldata signature
  112:     )
  113:         public
  114:         payable
  115:         retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
  116:     {
  117:         _swapAndExecute(
  118:             instructions.swapInstructions,
  119:             instructions.target,
  120:             instructions.paymentOperator,
  121:             instructions.payload,
  122:             instructions.refund
  123:         );
  124:     }
```

<img width="382" alt="image" src="https://github.com/code-423n4/2024-01-decent/assets/104318932/53d700dc-897a-4fe6-bff6-daa0147d9ada">

</br>
</br>

✅  The smart contract's `_getCallParams` function prepares the necessary parameters to be used during a bridge operation. Here, the calculation of the amount of gas allocated for the process plays an important role.

The amount of `100,000` gas determined by the `GAS_FOR_RELAY` constant is added to the gas need of the target chain specified by the `_dstGasForCall` parameter.

Layer 2 solutions in particular (such as Optimism, Base) are known for higher gas costs. If the total amount of gas allocated is not sufficient, the risk of reverting the transaction increases. This may cause the contract to fail to provide its expected functionality and users' transactions to fail.


```solidity

lib/decent-bridge/src/DecentEthRouter.sol:
   79  
   80:     function _getCallParams(
   81:         uint8 msgType,
   82:         address _toAddress,
   83:         uint16 _dstChainId,
   84:         uint64 _dstGasForCall,
   85:         bool deliverEth,
   86:         bytes memory additionalPayload
   87:     )
   88:         private
   89:         view
   90:         returns (
   91:             bytes32 destBridge,
   92:             bytes memory adapterParams,
   93:             bytes memory payload
   94:         )
   95:     {
   96:         uint256 GAS_FOR_RELAY = 100000; // <= @audit
   97:         uint256 gasAmount = GAS_FOR_RELAY + _dstGasForCall; // <= @audit
   98:         adapterParams = abi.encodePacked(PT_SEND_AND_CALL, gasAmount);
   99:         destBridge = bytes32(abi.encode(destinationBridges[_dstChainId]));
  100:         if (msgType == MT_ETH_TRANSFER) {
  101:             payload = abi.encode(msgType, msg.sender, _toAddress, deliverEth);
  102:         } else {
  103:             payload = abi.encode(
  104:                 msgType,
  105:                 msg.sender,
  106:                 _toAddress,
  107:                 deliverEth,
  108:                 additionalPayload
  109:             );
  110:         }
  111:     }

```



Recommended Mitigation Step ; Converting the `GAS_FOR_RELAY` value to a dynamic structure instead of a fixed value. Allowing users or contract administrators to update this value taking into account gas costs specific to different networks.


```diff
function _getCallParams(
    uint8 msgType,
    address _toAddress,
    uint16 _dstChainId,
    uint64 _dstGasForCall,
    bool deliverEth,
    bytes memory additionalPayload
)
    private
    view
    returns (
        bytes32 destBridge,
        bytes memory adapterParams,
        bytes memory payload
    )
{
-   uint256 GAS_FOR_RELAY = 100000;
+   uint256 GAS_FOR_RELAY = getGasForRelay(_dstChainId);
    uint256 gasAmount = GAS_FOR_RELAY + _dstGasForCall;
    adapterParams = abi.encodePacked(PT_SEND_AND_CALL, gasAmount);
    destBridge = bytes32(abi.encode(destinationBridges[_dstChainId]));
    if (msgType == MT_ETH_TRANSFER) {
        payload = abi.encode(msgType, msg.sender, _toAddress, deliverEth);
    } else {
        payload = abi.encode(
            msgType,
            msg.sender,
            _toAddress,
            deliverEth,
            additionalPayload
        );
    }
}

+ function getGasForRelay(uint16 _dstChainId) private view returns (uint256) {

+     if (_dstChainId == 1) { // Ethereum Mainnet
+         return 200000; 
+     } else if (_dstChainId == 56) { // BSC
+         return 150000; 
+     }
+     // default
+     return 100000;
+ }

```
</br>
</br>


✅  To implement ERC-2612, you would need to update the bridge function as follows for gas saving

```solidity
// Parameters required for ERC-2612 permit function
struct PermitParams {
    uint256 nonce;
    uint256 deadline;
    uint8 v;
    bytes32 r;
    bytes32 s;
}

function bridge(
    uint256 amt2Bridge,
    // ... (other parameters)
    PermitParams calldata permitParams // New parameter
) public payable onlyUtb {
    // ... (existing code)

    if (!gasIsEth) {
        // Permit call
        IERC2612(bridgeToken).permit(
            msg.sender,
            address(this),
            amt2Bridge,
            permitParams.deadline,
            permitParams.v,
            permitParams.r,
            permitParams.s
        );

        // Transfer call
        IERC20(bridgeToken).transferFrom(
            msg.sender,
            address(this),
            amt2Bridge
        );
    }

    // ... (remaining code)
}
```



### Time spent:
22 hours