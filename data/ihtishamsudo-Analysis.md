## Decent Protocol's Analysis Report 
### Preface 
This report is a summary of the analysis of Decent Protocol's Codebase.
- This report is not an extension of the documents provided by Decent Protocol.
- This report provides a high-level overview of the codebase and its security implications and some edge cases missed by developers and some suggestions to improve the codebase.
- This report aims to provide value to the sponsors and the developers of Decent Protocol.

**Index of table**
| Sl.no  | Particulars  |
|---|---|
| 1   | Overview  |
| 2  | Architecture view(Diagram) With Brief Description  |
| 3  | Approach taken in evaluating the codebase  |
| 4  | Centralization risks  |
| 5  | Security Consideration  |
| 6  | What protocol Did Unique  |
| 7  | Improvement   |
| 8  | Hours spend  |

## 1. Overview

Decent streamlines cross-chain transactions with a single click, utilizing UTB for routing and Decent's custom bridge. The swappable functions and adaptable bridgeAdapters enable seamless execution across different chains, supporting versatile transactions.

## 2. Architecture view(Diagram) With Brief Description

### 2.1 UTB Contracts 

The UTB contract invokes one of two methods: `swapAndExecute` or `bridgeAndExecute`. The swapAndExecute function facilitates transactions, enables cross-chain transactions for a user.
Collects fees and execute transactions on different L2 Chains.

<a href="https://imgur.com/eZsrXyJ"><img src="https://i.imgur.com/eZsrXyJ.png" title="source: imgur.com" /></a>

### 2.2 Swapper Contracts

Swapper perform swapNoPath function handles token swaps when no specific path of tokens to swap through is provided. The swapExactIn function executes a swap where a specific amount of input tokens are swapped for as many output tokens as possible, while the swapExactOut function performs a swap where a specific amount of output tokens are swapped for as few input tokens as possible.

<a href="https://imgur.com/evbvxHb"><img src="https://i.imgur.com/evbvxHb.png" title="source: imgur.com" /></a>

### 2.3 Bridge Adapter Contracts

Bridge Adapter Contracts does all the cross-chain transactions. The bridge adapter contract is a contract that is deployed on each chain that the UTB supports. The bridge adapter contract is responsible for receiving tokens from the UTB contract and sending them to the bridge contract on the other chain.

<a href="https://imgur.com/PwMVHa2"><img src="https://i.imgur.com/PwMVHa2.png" title="source: imgur.com" /></a>

### 2.4 Decent Bridge Contracts

Decent Bridge Contract includes cross-chain operations of `ETH` & `WETH` while also facilitates in executing transaction of `ETH` & `WETH`.

<a href="https://imgur.com/EWcAi6o"><img src="https://i.imgur.com/EWcAi6o.png" title="source: imgur.com" /></a>

### 2.5 Overall Architecture Of Codebase 

How all the contracts in Codebase (Scope) are interacting in the system.

<a href="https://imgur.com/SsGAQH2"><img src="https://i.imgur.com/SsGAQH2.png" title="source: imgur.com" /></a>
 
## 2.3 Approach taken in evaluating the codebase

Day 1: 
- Joined Contest and Cloned The Repository

- Setting up the environment for the codebase was pretty hectic most of the test cases were broken somehow (atleast on my end and many other wardens)

- Skimmed Over the Scope Contracts Specifically UTB contracts

- Manually Review the contracts and try to understand the flow of the codebase as much as possible

Day 2: 
- Couldn't find much time to review the codebase due to time constrains and other commitments

- But was able to Review Docs of Codebase 

Day 3: 

- Manual Deep Dive in the contracts to find weak spots and edge cases

- Able to find many edge cases which I will be discussing in the `imporvement` section

- Applied the BlackBoxing Technique, which is a technique to find bugs and errors by playing with the existing tests in test suite set up by the developers.

- Able to find bugs and recommendations by blackboxing but not strong to submit as a Med/High severity issue.

Day 4: 

- As Day 4 was an extended day, so Got some time to understand the Codebase at deep level to get a bigger picture of how protocol works so that I can write a better analysis report.

- Put audit tags on weak spots, Wrong Developer assumptions, wrongly written test cases, and many more.

- Connect the dots of the audit tags to write a better `improvement` section in analysis report to give developers a clean and clear picture of what needs to be improved.

## 2.4 Centralization risks

Codebase Contains Mainy Centralisation Risk that can brick the whole protocol if compromised.

### 2.4.1 Centralization Risk 1

`UTBOwned` 

All the UTB Contracts inheriting UTBOwned contracts which contains modifier `onlyutb` and is implemented in all the UTB contracts i.e   `UTB.sol`, `UTBFeeCollector.sol`, `UTBFeeCollector.sol`.

Modifier is initialized by `OnlyOwner`. If this address is compromise anyhow it will brick main functionality of codebase.

### 2.4.2 Centralization Risk 2

`bridge_adapters` 

Same technique is applied in bridge adapters that base contracts is inherited in every other `adapter` contrats which implement modifier for crucial functions in Adapter contracts and that too rely on OnlyOwner. 

Compromised `onlyOwner` Will Brick this functionality as well.

### 2.4.3 Centralization Risk 3

`decent_bridge`

Functions listed below have `onlyOwner()` and will land on risk and can damage the protocol in worse manner if owner address is compromised by various factors.

[function(mint)](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L24)

[function(burn)](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L28) 

[function(mintByOwner)](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L32)

[function(burnByOwner)](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L36)

## 2.5 Security Consideration

Unsuccessfull in running `forge coverage` command because of the broken codebase environment to get the exact codebase coverage.

However, having a look at the test case, it seems the the coverage was about `60-70%`.

Protocol used foundry for testing the codebase which is a good practice but the test cases were not written properly and many edge cases were missed by developers.

- Lack of 100% test cases covergae 

As there is only 60-70% test coverage so many test cases were missed by developers.

- Lack of Fuzz Testing 

As protocol have used foundry for testing, protocol must have done fuzzing to find unique edge cases and bugs, but it seems to be missing in the codebase.

- Lack of Invariant Testing 

As there is no invariant testing in the codebase, so it is hard to find if protocol is working as expected or not in different scenarios.


## 2.6 What protocol Did Unique

- Protocol vision itself is unique implementing one chain transaction over multiple L2 blockchain which will automatically reduce the hassle of moving funds from one chain to another chain to perform a specific transaction (buying an NFT, etc).

- Protocol is using `UTB` which is a unique concept in itself and is not used by any other protocol.

- Another thing I specifically want to mention is the unique ways of using Modifier `retrieveAndCollectFees` in `UTB.sol`. Seen this for the first time in any protocol it can be tricky at some points as more complex modifier will lead to more complex issues.

## 2.7 Improvement

- Not using `OZ ECDSA` for signature verification

As protocol is using signature in `collectingFees` function but using ecrecover which is not a good practice as it can lead malleability issues. 
Protocol should integereate `OZ ECDSA` for signature verification.

- Protocol missing require and checks 

Protocol is not bounding functionality to limits and missing require and checks in many places which can lead to many issues.

Not bounding certain functions to limits will cause many edge cases to pop up. 

Like, In One of the crucial function `swapExactIn` in `UniSwapper.sol` there is no check for minimum `amountIn` it means a user can swap any minimum amount as `1 WEI` possible.

``` solidity
    function swapExactIn(
        SwapParams memory swapParams, // SwapParams is a struct
        address receiver
    ) public payable routerIsSet returns (uint256 amountOut) {
        swapParams = _receiveAndWrapIfNeeded(swapParams);

        IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
            .ExactInputParams({
                path: swapParams.path,
                recipient: address(this),
                amountIn: swapParams.amountIn, //no check for minimum amount
                amountOutMinimum: swapParams.amountOut
            });

        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
        amountOut = IV3SwapRouter(uniswap_router).exactInput(params);

        _sendToRecipient(receiver, swapParams.tokenOut, amountOut);
    }
```

As it can result in `precision error` because of the limited number of decimal places in solidity. 

As anyone can do `spam transactions` with minimum amount possible which could lead to unneccessary load on Protocol Contracts.

Further improvements can be done by bounding functionalities of the codebase to restrict unexpected scenarios.

## 2.8 Hours spend

Total of 10-12 hours spend on the codebase.

### Time spent:
10 hours