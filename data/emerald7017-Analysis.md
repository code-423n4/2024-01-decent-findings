# Decent
Decent enables one-click transactions using any token across chains.

Architecture review of the cross-chain transaction protocol contracts.

**Modularity**

- The core UTB router contract is well modularized to interact with various swappers and bridge adapters via common interfaces (ISwapper, IBridgeAdapter). This enables flexibility.

- Although swappable components help iterate, excessive redundancy should be avoided.

**Separation of Concerns**

- There is clear separation between the routing (UTB), swapping (UniswapV2Swapper), bridging (DecentBridgeAdapter) and execution (UTBExecutor) responsibilities.

- Well defined roles reduce intersecting functionality leading to smaller attack surface.

**Trust Boundaries**

- By isolating external contract integration to specific adapter modules, the overall trust boundary remains narrow for the protocol's core logic.

- However verifying behavior of connected Chain A's impact on Chain B's state poses challenges.

**Simplicity**

- There is opportunity to reduce complexity further by minimizing custom swap, bridge and execution logic to only essentials.

- Stack usage, contract inheritance lineages should be Lean.

**Recommendations** 

- Consider on-chain credentials to govern approved swappers and adapters instead of contract owner.

- Formal verification of core protocol invariants using mathematical proofs can validate crux properties.

## Admin Abuse Risks
**Owner Role**

- The Owner role is assigned to the address that deploys the core contracts like `UTB.sol`, `DecentBridgeAdapter.sol` etc.

- The Owner has privileges to add/remove swappers and bridge adapters that plug into the UTB router contract. **This poses risk if malicious contracts are added.**

- **Recommendation**: Make the Owner role a multi-sig wallet contract to prevent centralized abuse. Implement a DAO based governance model to control admin changes.

**Fee Collector** 

- UTBFeeCollector contract collects protocol fees on UTB trades. Owner can withdraw fees. 

- There is no max limit enforced for fee amounts set. And fees are variable based on trade.

- Risk of profiting unfairly from user trades.

**Bridge Operator**

- Has ability to pause bridge operations and trigger emergency withdrawals if bridge funds are compromised.

- Checks needed to ensure this capability not abused to prevent asset transfers.

## Dangerous admin functions that enable full control

**Asset Transfers**

- The `UTB.sol` contract contains an `adminWithdraw` function that allows the contract owner to drain assets from the contract.
- This presents the risk of admin extracting user funds during swap or bridge operations.

- Similarly the `UTBFeeCollector` allows owner to arbitrarily withdraw accumulated fees. No maximum cap on amount.

**Contract Upgrades**

- The bridge adapters like `DecentBridgeAdapter.sol` do not appear to have any explicit contract upgrade mechanisms.

- The base `UTB.sol` router similarly does not support replacing critical connected contracts.

**Recommendations**

- Remove unnecessary admin withdrawal capabilities. Implement maximum withdrawal caps based on accumulated fees.

- Consider adopting the UUPS (Universal Upgradeable Proxy Standard) pattern for smooth contract migrations.

## Access Control Analysis
**Owner Role Checks**

- The base contracts like `UTB.sol`, `UTBFeeCollector.sol` are all assigned the deploying address as the immutable Owner.

- There are no additional access control mechanisms like multi-sig or time-locks in place currently.

- This means the Owner has unilateral control of critical admin functions.

**Multi-sig Usage**

- Admin capabilities like adding/removing swappers in UTB, or emergency bridge pause are not protected by multi-sig.

- The admin changes are subject to the sole Owner address' transactions.

- This increases risk of malicious actions by a single entity.

**Time-lock Gaps** 

- No admin changes are rate-limited by time-locks. They can execute instantly based on Owner address.

- This does not allow community intervention for controversial changes.

**Recommendations**

- Implement multi-sig wallet for Owner address to distribute trust.

- Apply minimum time-lock periods for proposing and executing admin changes.


## Social recovery mechanisms in place for critical admin functions
**Social Recovery Review**

- There do not seem to be any explicit social recovery schemes implemented yet.

- Critical functions like adding swap/bridge contracts, emergency pausing, fee withdrawals are solely controlled by the Owner address.

- There are no additional approvers required currently outside of the Owner admin key.

**Centralization Risks**

- This centralized control poses a risk of malicious actions or fund extraction by a single party.

- Lack of social recovery means limited recourse for the community if the admin key is compromised.

**Recommendations**

- Implement community guardian addresses (set of multisig) for social recovery of critical admin functions.

- Emergency pause of bridge could need approval from multiple bridge entities to prevent unilateral abuse. 

- Fee withdrawal rates could be votable by the DAO community or protocol users.

Adding social recovery mechanisms via multi-party approvals or DAO oversight would help mitigate centralization of power or abuse risks.

## Access controls around the emergency and circuit breaker functionality
**Circuit Breaker Checks**

- There do not seem to be explicit circuit breaker mechanisms in the contracts.

- If bridge was compromised, Owner could unilaterally pause it blocking withdrawals.

**Risks**

- Sole reliance on Owner admin key for emergency functions increases potential for abuse of power.

**Recommendations**

- Require multi-party consent via an on-chain DAO vote to invoke emergency functions like pause.

- Implement automated circuit breakers that halt critical operations based on quantified risk metrics and thresholds.

- Assign multi-sig Schemes or time-locks to manual circuit breakers to distribute trust.

## Systemic Risks
Analysis of systemic risks introduced through integration with bridges and external protocols.

**Trust Dependencies**

- The DecentBridgeAdapter relies on the security and availability of LayerZero's bridge infrastructure. Any compromises or outages of LayerZero could paralyze cross-chain transactions.

- The UniswapV2Swapper depends on the AMM liquidity pools continuing to function and provide accurate prices.

**Inherited Risk Factors** 

- Issues like transaction censorship, liquidity shortages, price oracle manipulations on LayerZero or Uniswap could propagate risk.

- Smart contract risks present in LayerZero and Uniswap like reentrancy bugs can threaten integrated security posture.

**Indirect Impacts**

- Asset transfers may fail despite the base protocol functioning correctly due to these external dependencies being compromised. 

- Could lead to broken user experiences and loss of trust in the system.

**Recommendations** 

- Implement decoupling layers, redundancy and modular replaceability where feasible to limit ripple effects of external issues.

## Technical Risks
**Error Handling**

- Inadequate handling of failed swaps, erroneous transfers or reverted executions could result in fund locking or losses. 

- Extensive unit and integration test coverage required for various error scenarios - failed transfers, insufficient allowances, gas limits etc.

- Usage of try/catch blocks for custom errors and checks for revert reasons.

**Arithmetic Bugs**

- Overflow risks exist in fee calculations, token transfer amounts, liquidity balances.

- Use OpenZeppelin SafeMath libraries to mitigate over/underflows. Validate return values.

**MEV Resistance** 

- Transaction ordering dependence and public calldata allows front-running attacks during swaps and contract interactions.

- Consider usage of commit-reveal schemes to hide sensitive parameters before execution. 

- Orderbook model can reduce MEV potential during token swaps.

## Integration Risks
**Upgradeability Issues**

- Upgrades to LayerZero bridge or changes to Uniswap V2 contracts can break functionality reliant on specific method call schemas.

- Implement proxy contracts and registries to make critical connectors like DecentBridgeAdapter upgradeable in a non-breaking manner. 

- Adopt EIP-1967 and transparent proxies to smoothly migrate contracts under interfaces.

**Performance Bottlenecks** 

- Congestion, throughput drops or outages on LayerZero and target chains can cause transaction bottlenecks — failed settlements, stuck funds.

**Dependency Bloat**

- Each added external contract dependency increases potential attack surface and points of failure.

- Avoid integrating additional contracts without proper due diligence. Favor implementing necessary base functions directly where feasible.

- Enforce version pinning of external contract dependencies to avoid surprise breakages.

### Time spent:
21 hours