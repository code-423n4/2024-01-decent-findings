# Purpose of Analysis
The purpose of this analysis is to give an overview of the vulnerabilities found, aggregate them into common issues that surfaced into the protocol as well as to provide recommendation to prevent further mistakes.

# Summary of Key Findings

1. **Missing access control check can allow anyone to mint or burn DcntEth from anyone**: The `setRouter` function in `DcntEth.sol` did not have access control checks and therefore anyone could set themselves as the router and mint or burn DcntEth OFTv2 tokens from any account.

2. **Anyone can bypass fees in swapAndExecute using receiveFromBridge**: The `receiveFromBridge` function in `UTB.sol` did not have any access control checks to ensure the `msg.sender` was a bridge adapter. This function allowed users to call `_swapAndExecute` without paying fees.

3. **Anyone can send cross-chain transactions without paying fees.**: Anyone can call `bridge` and `bridgeWithPayload` to perform cross-chain transactions in `DecentEthRouter.sol`. These functions are intermediate functions in the bridge process meant only to be called by the bridge adapters. Therefore, this allowed users to bypass the fees in `UTB.sol`.

4. **If the call destination bridge executor makes fails, then funds will not get refunded to the correct address on the destination chain**: If the call from the bridge executor fails, then the tokens will get refunded to the address of the source chain's bridge adapter on the destination chain, which is not owned by either the user performing the transaction or Decent.

5. **Users will lose their cross-chain transaction if the destination router do not have enough WETH reserves.**: If there are not enough WETH reserves to support an incoming cross-chain transaction, the dcntEth token will be sent to the `to` address. However, in almost all cases, the `to` address is the address of the destination bridge adapter which cannot refund the tokens to the user.

More details can be found in the individual reports I made.

# Common Issues

1. **Missing Access Control**: Findings 1, 2 and 3 surface from the same mistake of missing access control on key functions.
**Recommendation**: It is important that the team reviews privileged state-changing functions for access control. The team should also trace out the functions being executed during a bridge process and ensure all of them have adequate access control checks to prevent users from bypassing fees.

2. **Funds send to the wrong address**: Findings 4 and 5 surface from the wrong address being used when sending funds. 
**Recommendation**: The team should review that the recipient of the funds, especially refunds when the transaction fails halfway, is always the user's address or an address specifically provided by the user for refunds.

# Further Issues

**The incentive mechanism to provide WETH reserves for the DecentEthRouter is not in-scope and therefore has not been reviewed**: 

When a user wants to send a cross chain transaction, UTB will perform a series of steps (not important) before sending in ETH / WETH into the source router. 

The source router then stores the ETH / WETH into the WETH reserves and sends the dcntEth from the source router's dcntEth reserves to the destination router.

The destination router receives and stores the dcntEth into its reserves and sends the ETH / WETH to the destination bridge executor.

If there are little to no WETH reserves, there are 2 scenarios which can occur: 

1) If a router receives more cross-chain transactions than sending cross-chain transactions, then on a receiving cross-chain transaction, there will be no WETH reserves to send to the bridge executor which will cause the bridge execution to fail halfway. 

2) If a router sends more cross-chain transactions than receiving cross-chain transactions, then the router cannot send a cross-chain transaction because it will not have enough dcntEth reserve to send. This is because dcntEth is only minted to the router's reserves when a user deposits ETH / WETH via `addLiquidityEth` or `addLiquidityWeth` and not when the router receives the WETH / ETH from the bridge adapter.

As such, for the continued functionality of the bridge there must be some WETH reserves deposited into the router by users

When clarifying with the protocol team, they mentioned the following:

> assume third parties are incentivized to provide reserve with offchain contracts

These incentive mechanisms is out-of-scope and have not been reviewed in this audit.

**Recommendation**: These incentive contracts should be either reviewed in another audit or reviewed carefully by the protocol team to ensure that these contracts function as intended and that users will be incentivised to place WETH to the reserves as these WETH reserves are crucial for the stability and functionality of the bridge.

### Time spent:
10 hours