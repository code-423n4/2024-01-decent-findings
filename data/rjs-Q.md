> Please note: this report has been reviewed to ensure it *does not* contain automated findings from the bot race winner or 4naly3er

# Summary

## Low Issues
| # | Title | Instances |
| - | - | - |
| L-01 | [Best Practice: Follow the Check-Effects-Interaction pattern](#l-01-best-practice-follow-the-check-effects-interaction-pattern) | 10 |
| L-02 | [Missing checks for `address(0x0)` in the constructor/initializer](#l-02-missing-checks-for-address0x0-in-the-constructorinitializer) | 1 |
| L-03 | [Missing checks for `ecrecover()` signature malleability](#l-03-missing-checks-for-ecrecover-signature-malleability) | 1 |
| L-04 | [Missing checks for state variable assignments](#l-04-missing-checks-for-state-variable-assignments) | 11 |
| L-05 | [Solidity version 0.8.20 may not work on other chains due to `PUSH0`](#l-05-solidity-version-0820-may-not-work-on-other-chains-due-to-push0) | 11 |
| L-06 | [Unsafe solidity low-level call can cause gas grief attack](#l-06-unsafe-solidity-low-level-call-can-cause-gas-grief-attack) | 3 |
| L-07 | [`call()`/`delegatecall()` has unchecked return value](#l-07-calldelegatecall-has-unchecked-return-value) | 2 |
| L-08 | [`call()`/`delegatecall()` should check for contract-existence](#l-08-calldelegatecall-should-check-for-contract-existence) | 7 |
| L-09 | [`call()`/`delegatecall()` should specify a gas limit](#l-09-calldelegatecall-should-specify-a-gas-limit) | 7 |

## Non-Critical Issues
| # | Title | Instances |
| - | - | - |
| N-01 | [Assembly blocks should have extensive comments](#n-01-assembly-blocks-should-have-extensive-comments) | 1 |
| N-02 | [Complex arithmetic expression](#n-02-complex-arithmetic-expression) | 2 |
| N-03 | [Consider adding a block/deny-list](#n-03-consider-adding-a-blockdeny-list) | 1 |
| N-04 | [Consider moving `msg.sender` checks to `modifier`s](#n-04-consider-moving-msgsender-checks-to-modifiers) | 3 |
| N-05 | [Consider using descriptive `constant`s when passing zero as a function argument](#n-05-consider-using-descriptive-constants-when-passing-zero-as-a-function-argument) | 1 |
| N-06 | [Consider using modifiers for access control](#n-06-consider-using-modifiers-for-access-control) | 3 |
| N-07 | [Consider using named function call parameters when `>= 4` parameters](#n-07-consider-using-named-function-call-parameters-when--4-parameters) | 28 |
| N-08 | [Duplicate strings defined](#n-08-duplicate-strings-defined) | 1 |
| N-09 | [Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, for readability](#n-09-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct-for-readability) | 2 |
| N-10 | [NatSpec: public state variable declarations should have @notice tag](#n-10-natspec-public-state-variable-declarations-should-have-notice-tag) | 25 |
| N-11 | [NatSpec: state variable declarations should have @dev tag](#n-11-natspec-state-variable-declarations-should-have-dev-tag) | 39 |
| N-12 | [Redundant `else` block](#n-12-redundant-else-block) | 2 |
| N-13 | [State variables should include comments](#n-13-state-variables-should-include-comments) | 39 |
| N-14 | [Syntax: variables need not be initialized to zero](#n-14-syntax-variables-need-not-be-initialized-to-zero) | 1 |
| N-15 | [Typos](#n-15-typos) | 1 |
| N-16 | [Unnecessary cast](#n-16-unnecessary-cast) | 1 |
| N-17 | [`abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`](#n-17-abiencodepacked-should-not-be-used-with-dynamic-types-when-passing-the-result-to-a-hash-function-such-as-keccak256) | 1 |
| N-18 | [`require()`/`revert()` statements should have descriptive reason strings](#n-18-requirerevert-statements-should-have-descriptive-reason-strings) | 1 |

# Low Issues

## [L-01] Best Practice: Follow the Check-Effects-Interaction pattern

Code should follow the best-practice of [check-effects-interaction](https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-11-coding-patterns/topic/checks-effects-interactions/), where state variables are updated before any external calls are made. Doing so prevents a large class of reentrancy bugs.

*Instances: 10*

```solidity
File: src/UTB.sol

     |  // @audit-info External call
  76 |  wrapped.deposit{value: swapParams.amountIn}();
     |  // @audit-issue State variable assignment after external call
  77 |  swapParams.tokenIn = address(wrapped);
     |  // @audit-issue State variable assignment after external call
  78 |  swapInstructions.swapPayload = swapper.updateSwapParams(
  79 |      swapParams,
  80 |      swapInstructions.swapPayload
  81 |  );

     |      // @audit-info External call
  83 |      IERC20(swapParams.tokenIn).transferFrom(
  84 |          msg.sender,
  85 |          address(this),
  86 |          swapParams.amountIn
  87 |      );
  :  |
     |  // @audit-info External call
  90 |  IERC20(swapParams.tokenIn).approve(
  91 |      address(swapper),
  92 |      swapParams.amountIn
  93 |  );
  :  |
     |  // @audit-issue State variable assignment after external call
  95 |  (tokenOut, amountOut) = swapper.swap(swapInstructions.swapPayload);
```
GitHub: [77](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L77), [78-81](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L78-L81), [95](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L95)


```solidity
File: src/UTBExecutor.sol

     |  // @audit-info External call
  61 |  IERC20(token).transferFrom(msg.sender, address(this), amount);
     |  // @audit-info External call
  62 |  IERC20(token).approve(paymentOperator, amount);
  :  |
     |      // @audit-issue State variable assignment after external call
  65 |      (success, ) = target.call{value: extraNative}(payload);
  :  |
     |      // @audit-issue State variable assignment after external call
  70 |      (success, ) = target.call(payload);
```
GitHub: [65](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L65), [70](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L70)


```solidity
File: src/swappers/UniSwapper.sol

     |      // @audit-info External call
  83 |      IERC20(swapParams.tokenIn).transferFrom(
  84 |          msg.sender,
  85 |          address(this),
  86 |          swapParams.amountIn
  87 |      );
  :  |
     |  // @audit-issue State variable assignment after external call
  90 |  swapParams.tokenIn = wrapped;

     |  // @audit-info External call
 137 |  IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
     |  // @audit-issue State variable assignment after external call
 138 |  amountOut = IV3SwapRouter(uniswap_router).exactInput(params);

     |  // @audit-info External call
 158 |  IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
     |  // @audit-issue State variable assignment after external call
 159 |  amountIn = IV3SwapRouter(uniswap_router).exactOutput(params);
```
GitHub: [90](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L90), [138](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L138), [159](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L159)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-info External call
 178 |  weth.deposit{value: _amount}();
     |  // @audit-issue State variable assignment after external call
 179 |  gasValue = msg.value - _amount;

     |  // @audit-info External call
 181 |  weth.transferFrom(msg.sender, address(this), _amount);
     |  // @audit-issue State variable assignment after external call
 182 |  gasValue = msg.value;
```
GitHub: [179](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L179), [182](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L182)


## [L-02] Missing checks for `address(0x0)` in the constructor/initializer

Since `address(0x0)` has no private key, it is almost always a mistake when used outside of burning operations.

*Instances: 1*

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue `_bridgeToken` should be checked against 0x0
  21 |  constructor(bool _gasIsEth, address _bridgeToken) BaseAdapter() {
```
GitHub: [21](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L21)


## [L-03] Missing checks for `ecrecover()` signature malleability

`ecrecover()` accepts as valid, two versions of signatures, meaning an attacker can use the same signature twice, or an attacker may be able to front-run the original signer with the altered version of the signature, causing the signer's transaction to revert due to nonce reuse. Consider adding checks for signature malleability, or using OpenZeppelin's `ECDSA` library to perform the extra checks necessary in order to prevent malleability.

*Instances: 1*

```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-issue Verify s or add a nonce
  53 |  address recovered = ecrecover(constructedHash, v, r, s);
```
GitHub: [53](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L53)


## [L-04] Missing checks for state variable assignments

There are some missing checks in these functions, and this could lead to unexpected scenarios. Consider always adding a sanity check for state variables.

*Instances: 11*

```solidity
File: src/UTB.sol

     |  // @audit-issue `swapper` is assigned to a state variable, unvalidated
 325 |  function registerSwapper(address swapper) public onlyOwner {

     |  // @audit-issue `bridge` is assigned to a state variable, unvalidated
 334 |  function registerBridge(address bridge) public onlyOwner {
```
GitHub: [325](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L325), [334](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L334)


```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-issue `_signer` is assigned to a state variable, unvalidated
  18 |  function setSigner(address _signer) public onlyOwner {
```
GitHub: [18](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L18)


```solidity
File: src/bridge_adapters/BaseAdapter.sol

     |  // @audit-issue `_executor` is assigned to a state variable, unvalidated
  19 |  function setBridgeExecutor(address _executor) public onlyOwner {
```
GitHub: [19](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L19)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue `decentBridgeAdapter` is assigned to a state variable, unvalidated
  34 |  function registerRemoteBridgeAdapter(
  35 |      uint256 dstChainId,
  36 |      uint16 dstLzId,
  37 |      address decentBridgeAdapter
  38 |  ) public onlyOwner {
```
GitHub: [34-38](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L34-L38)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue `_sgEth` is assigned to a state variable, unvalidated
  37 |  function setStargateEth(address _sgEth) public onlyOwner {

     |  // @audit-issue `decentBridgeAdapter` is assigned to a state variable, unvalidated
  45 |  function registerRemoteBridgeAdapter(
  46 |      uint256 dstChainId,
  47 |      uint16 dstLzId,
  48 |      address decentBridgeAdapter
  49 |  ) public onlyOwner {
```
GitHub: [37](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L37), [45-49](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L45-L49)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue `_router` is assigned to a state variable, unvalidated
  20 |  function setRouter(address _router) public onlyOwner {

     |  // @audit-issue `_wrapped` is assigned to a state variable, unvalidated
  24 |  function setWrapped(address payable _wrapped) public onlyOwner {
```
GitHub: [20](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L20), [24](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L24)


```solidity
File: lib/decent-bridge/src/DcntEth.sol

     |  // @audit-issue `_router` is assigned to a state variable, unvalidated
  20 |  function setRouter(address _router) public {
```
GitHub: [20](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L20)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue `_routerAddress` is assigned to a state variable, unvalidated
  73 |  function addDestinationBridge(
  74 |      uint16 _dstChainId,
  75 |      address _routerAddress
  76 |  ) public onlyOwner {
```
GitHub: [73-76](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L73-L76)


## [L-05] Solidity version 0.8.20 may not work on other chains due to `PUSH0`

> The `readme.md` specifically states L2s are in scope: `Check all that apply (e.g. timelock, NFT, AMM, ERC20, rollups, etc.): ... Uses L2`

The compiler for Solidity 0.8.20 switches the default target EVM version to [Shanghai](https://blog.soliditylang.org/2023/05/10/solidity-0.8.20-release-announcement/#important-note), which includes the new `PUSH0` op code. This op code may not yet be implemented on all L2s, so deployment on these chains will fail. To work around this issue, use an earlier [EVM](https://docs.soliditylang.org/en/v0.8.20/using-the-compiler.html?ref=zaryabs.com#setting-the-evm-version-to-target) [version](https://book.getfoundry.sh/reference/config/solidity-compiler#evm_version). While the project itself may or may not compile with 0.8.20, other projects with which it integrates, or which extend this project may, and those projects will have problems deploying these contracts/libraries.

*Instances: 11*

```solidity
File: src/UTB.sol

     |  // @audit-issue Require Solidity 0.8.19 or lower
   2 |  pragma solidity ^0.8.0;
```
GitHub: [2](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L2)


```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Require Solidity 0.8.19 or lower
   2 |  pragma solidity ^0.8.0;
```
GitHub: [2](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L2)


```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-issue Require Solidity 0.8.19 or lower
   2 |  pragma solidity ^0.8.0;
```
GitHub: [2](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L2)


```solidity
File: src/bridge_adapters/BaseAdapter.sol

     |  // @audit-issue Require Solidity 0.8.19 or lower
   2 |  pragma solidity ^0.8.0;
```
GitHub: [2](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L2)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue Require Solidity 0.8.19 or lower
   2 |  pragma solidity ^0.8.0;
```
GitHub: [2](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L2)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Require Solidity 0.8.19 or lower
   2 |  pragma solidity ^0.8.0;
```
GitHub: [2](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L2)


```solidity
File: src/swappers/SwapParams.sol

     |  // @audit-issue Require Solidity 0.8.19 or lower
   2 |  pragma solidity ^0.8.0;
```
GitHub: [2](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L2)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Require Solidity 0.8.19 or lower
   2 |  pragma solidity ^0.8.0;
```
GitHub: [2](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L2)


```solidity
File: lib/decent-bridge/src/DcntEth.sol

     |  // @audit-issue Require Solidity 0.8.19 or lower
   2 |  pragma solidity ^0.8.13;
```
GitHub: [2](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L2)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Require Solidity 0.8.19 or lower
   2 |  pragma solidity ^0.8.13;
```
GitHub: [2](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L2)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue Require Solidity 0.8.19 or lower
   2 |  pragma solidity ^0.8.0;
```
GitHub: [2](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L2)


## [L-06] Unsafe solidity low-level call can cause gas grief attack

Using the low-level calls of a solidity address can leave the contract open to gas grief attacks. These attacks occur when the called contract returns a large amount of data.
So when calling an external contract, it is necessary to check the length of the return data before reading/copying it (using `returndatasize()`).

*Instances: 3*

```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Check returndatasize()
  52 |  (success, ) = target.call{value: amount}(payload);

     |  // @audit-issue Check returndatasize()
  65 |  (success, ) = target.call{value: extraNative}(payload);

     |  // @audit-issue Check returndatasize()
  70 |  (success, ) = target.call(payload);
```
GitHub: [52](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L52), [65](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L65), [70](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L70)


## [L-07] `call()`/`delegatecall()` has unchecked return value

The function being called may revert, which will be indicated by the return value to `call()`/`delegatecall()`. If the return value is not checked, the code will continue on as if there was no error, rather than reverting with the error encountered.

*Instances: 2*

```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Check the return value of the call
  54 |  (refund.call{value: amount}(""));

     |  // @audit-issue Check the return value of the call
  67 |  (refund.call{value: extraNative}(""));
```
GitHub: [54](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L54), [67](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L67)


## [L-08] `call()`/`delegatecall()` should check for contract-existence

Low-level calls return success if there is no code present at the specified address. In addition to the zero-address checks, add a check to verify that `<address>.code.length > 0`.

*Instances: 7*

```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Add contract existence checks
  52 |  (success, ) = target.call{value: amount}(payload);

     |  // @audit-issue Add contract existence checks
  54 |  (refund.call{value: amount}(""));

     |  // @audit-issue Add contract existence checks
  65 |  (success, ) = target.call{value: extraNative}(payload);

     |  // @audit-issue Add contract existence checks
  67 |  (refund.call{value: extraNative}(""));

     |  // @audit-issue Add contract existence checks
  70 |  (success, ) = target.call(payload);
```
GitHub: [52](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L52), [54](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L54), [65](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L65), [67](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L67), [70](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L70)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue Add contract existence checks
  33 |  (bool success, ) = target.call(callPayload);

     |  // @audit-issue Add contract existence checks
  61 |  (bool success, ) = target.call{value: amount}(callPayload);
```
GitHub: [33](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L33), [61](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L61)


## [L-09] `call()`/`delegatecall()` should specify a gas limit

There is no limit specified on the amount of gas used, so the recipient can use up all of the transaction's gas, causing it to revert. Use `addr.call{gas: <amount>}("")` or [this](https://github.com/nomad-xyz/ExcessivelySafeCall) library instead.

*Instances: 7*

```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Specify { gas: <amount> } option
  52 |  (success, ) = target.call{value: amount}(payload);

     |  // @audit-issue Specify { gas: <amount> } option
  54 |  (refund.call{value: amount}(""));

     |  // @audit-issue Specify { gas: <amount> } option
  65 |  (success, ) = target.call{value: extraNative}(payload);

     |  // @audit-issue Specify { gas: <amount> } option
  67 |  (refund.call{value: extraNative}(""));

     |  // @audit-issue Specify { gas: <amount> } option
  70 |  (success, ) = target.call(payload);
```
GitHub: [52](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L52), [54](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L54), [65](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L65), [67](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L67), [70](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L70)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue Specify { gas: <amount> } option
  33 |  (bool success, ) = target.call(callPayload);

     |  // @audit-issue Specify { gas: <amount> } option
  61 |  (bool success, ) = target.call{value: amount}(callPayload);
```
GitHub: [33](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L33), [61](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L61)

# Non-Critical Issues

## [N-01] Assembly blocks should have extensive comments

Assembly blocks take a lot more time to audit than normal Solidity code, and often have gotchas and side-effects that the Solidity versions of the same code do not. Consider adding more comments explaining what is being done in every step of the assembly code, and describe why assembly is being used instead of Solidity.

*Instances: 1*

```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-issue Inline assembly should be heavily commented
  31 |  assembly {
  32 |      r := mload(add(signature, 32))
  33 |      s := mload(add(signature, 64))
  34 |      v := byte(0, mload(add(signature, 96)))
  35 |  }
```
GitHub: [31-35](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L31-L35)


## [N-02] Complex arithmetic expression

To maintain readability in code, particularly in Solidity which can involve complex mathematical operations, it is often recommended to limit the number of arithmetic operations to a maximum of 2-3 per line. Too many operations in a single line can make the code difficult to read and understand, increase the likelihood of mistakes, and complicate the process of debugging and reviewing the code. Consider splitting such operations over more than one line, take special care when dealing with division however. Try to limit the number of arithmetic operations to a maximum of 3 per line.

*Instances: 2*

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Simplify by using intermediate variables
  66 |  return (amt2Bridge * (1e4 - SG_FEE_BPS)) / 1e4;

     |  // @audit-issue Simplify by using intermediate variables
 170 |  router.swap{value: msg.value}(
 171 |      getDstChainId(additionalArgs), //lzBridgeData._dstChainId, // send to LayerZero chainId
 172 |      getSrcPoolId(additionalArgs), //lzBridgeData._srcPoolId, // source pool id
 173 |      getDstPoolId(additionalArgs), //lzBridgeData._dstPoolId, // dst pool id
 174 |      refund, // refund adddress. extra gas (if any) is returned to this address
 175 |      amt2Bridge, // quantity to swap
 176 |      (amt2Bridge * (10000 - SG_FEE_BPS)) / 10000, // the min qty you would accept on the destination, fee is 6 bips
 177 |      getLzTxObj(additionalArgs), // additional gasLimit increase, airdrop, at address
 178 |      abi.encodePacked(getDestAdapter(dstChainId)),
 179 |      bridgePayload // bytes param, if you wish to send additional payload you can abi.encode() them here
 180 |  );
```
GitHub: [66](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L66), [170-180](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L170-L180)


## [N-03] Consider adding a block/deny-list

Doing so will significantly increase centralization, but will help to prevent hackers from using stolen tokens.

*Instances: 1*

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Consider adding a block/deny-list
  12 |  contract DecentEthRouter is IDecentEthRouter, IOFTReceiverV2, Owned {
```
GitHub: [12](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L12)


## [N-04] Consider moving `msg.sender` checks to `modifier`s

Using `modifier`s makes the contract guard intention of the code more clear, and also guarantees no other code can run before the modifier runs.

*Instances: 3*

```solidity
File: src/bridge_adapters/BaseAdapter.sol

     |  // @audit-issue Refactor into a modifier
  12 |  require(
  13 |      msg.sender == address(bridgeExecutor),
  14 |      "Only bridge executor can call this"
  15 |  );
```
GitHub: [12-15](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L12-L15)


```solidity
File: lib/decent-bridge/src/DcntEth.sol

     |  // @audit-issue Refactor into a modifier
   9 |  require(msg.sender == router);
```
GitHub: [9](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L9)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Refactor into a modifier
  43 |  require(
  44 |      address(dcntEth) == msg.sender,
  45 |      "DecentEthRouter: only lz App can call"
  46 |  );
```
GitHub: [43-46](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L43-L46)


## [N-05] Consider using descriptive `constant`s when passing zero as a function argument

Passing `0` or `0x0` as a function argument can sometimes result in a security issue(e.g. passing zero as the slippage parameter). A historical example is the infamous `0x0` address bug where numerous tokens were lost. This happens because `0` can be interpreted as an uninitialized `address`, leading to transfers to the `0x0` `address`, effectively burning tokens. Moreover, `0` as a denominator in division operations would cause a runtime exception. It's also often indicative of a logical error in the caller's code.

Consider using a constant variable with a descriptive name, so it's clear that the argument is intentionally being used, and for the right reasons.

*Instances: 1*

```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Replace literal 0 with an explicit constant
  28 |  execute(target, paymentOperator, payload, token, amount, refund, 0);
```
GitHub: [28](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L28)


## [N-06] Consider using modifiers for access control

Modifiers in Solidity can improve code readability and modularity by encapsulating repetitive checks, such as address validity checks, into a reusable construct. For example, an `onlyOwner` modifier can be used to replace repetitive `require(msg.sender == owner)` checks across several functions, reducing code redundancy and enhancing maintainability. To implement, define a modifier with the check, then apply the modifier to relevant functions.

*Instances: 3*

```solidity
File: src/bridge_adapters/BaseAdapter.sol

     |  // @audit-issue Refactor into a modifier
  12 |  require(
  13 |      msg.sender == address(bridgeExecutor),
  14 |      "Only bridge executor can call this"
  15 |  );
```
GitHub: [12-15](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L12-L15)


```solidity
File: lib/decent-bridge/src/DcntEth.sol

     |  // @audit-issue Refactor into a modifier
   9 |  require(msg.sender == router);
```
GitHub: [9](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L9)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Refactor into a modifier
  43 |  require(
  44 |      address(dcntEth) == msg.sender,
  45 |      "DecentEthRouter: only lz App can call"
  46 |  );
```
GitHub: [43-46](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L43-L46)


## [N-07] Consider using named function call parameters when `>= 4` parameters

Named function calls in Solidity greatly improve code readability by explicitly mapping arguments to their respective parameter names. This clarity becomes critical when dealing with functions that have numerous or complex parameters, reducing potential errors due to misordered arguments. Therefore, adopting named function calls contributes to more maintainable and less error-prone code. The following findings are for function calls with 4 or more praameters.

*Instances: 28*

```solidity
File: src/UTB.sol

     |  // @audit-issue Consider named parameters
 117 |  _swapAndExecute(
 118 |      instructions.swapInstructions,
 119 |      instructions.target,
 120 |      instructions.paymentOperator,
 121 |      instructions.payload,
 122 |      instructions.refund
 123 |  );

     |  // @audit-issue Consider named parameters
 143 |  executor.execute{value: amountOut}(
 144 |      target,
 145 |      paymentOperator,
 146 |      payload,
 147 |      tokenOut,
 148 |      amountOut,
 149 |      refund
 150 |  );

     |  // @audit-issue Consider named parameters
 153 |  executor.execute(
 154 |      target,
 155 |      paymentOperator,
 156 |      payload,
 157 |      tokenOut,
 158 |      amountOut,
 159 |      refund
 160 |  );

     |  // @audit-issue Consider named parameters
 289 |  IBridgeAdapter(bridgeAdapters[instructions.bridgeId]).bridge{
 290 |      value: bridgeFee + (native ? amt2Bridge : 0)
 291 |  }(
 292 |      amt2Bridge,
 293 |      instructions.postBridge,
 294 |      instructions.dstChainId,
 295 |      instructions.target,
 296 |      instructions.paymentOperator,
 297 |      instructions.payload,
 298 |      instructions.additionalArgs,
 299 |      instructions.refund
 300 |  );

     |  // @audit-issue Consider named parameters
 318 |  _swapAndExecute(postBridge, target, paymentOperator, payload, refund);
```
GitHub: [117-123](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L117-L123), [143-150](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L143-L150), [153-160](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L153-L160), [289-300](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L289-L300), [318](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L318)


```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Consider named parameters
  28 |  execute(target, paymentOperator, payload, token, amount, refund, 0);
```
GitHub: [28](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L28)


```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-issue Consider named parameters
  53 |  address recovered = ecrecover(constructedHash, v, r, s);
```
GitHub: [53](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L53)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue Consider named parameters
  56 |  router.estimateSendAndCallFee(
  57 |      router.MT_ETH_TRANSFER_WITH_PAYLOAD(),
  58 |      lzIdLookup[dstChainId],
  59 |      target,
  60 |      swapParams.amountIn,
  61 |      dstGas,
  62 |      false,
  63 |      payload
  64 |  );

     |  // @audit-issue Consider named parameters
 117 |  router.bridgeWithPayload{value: msg.value}(
 118 |      lzIdLookup[dstChainId],
 119 |      destinationBridgeAdapter[dstChainId],
 120 |      swapParams.amountIn,
 121 |      false,
 122 |      dstGas,
 123 |      bridgePayload
 124 |  );

     |  // @audit-issue Consider named parameters
 147 |  IUTB(utb).receiveFromBridge(
 148 |      postBridge,
 149 |      target,
 150 |      paymentOperator,
 151 |      payload,
 152 |      refund
 153 |  );
```
GitHub: [56-64](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L56-L64), [117-124](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L117-L124), [147-153](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L147-L153)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Consider named parameters
  81 |  bridgePayload = abi.encode(
  82 |      postBridge,
  83 |      target,
  84 |      paymentOperator,
  85 |      payload,
  86 |      refund
  87 |  );

     |  // @audit-issue Consider named parameters
  91 |  callBridge(
  92 |      amt2Bridge,
  93 |      dstChainId,
  94 |      bridgePayload,
  95 |      additionalArgs,
  96 |      refund
  97 |  );

     |  // @audit-issue Consider named parameters
 170 |  router.swap{value: msg.value}(
 171 |      getDstChainId(additionalArgs), //lzBridgeData._dstChainId, // send to LayerZero chainId
 172 |      getSrcPoolId(additionalArgs), //lzBridgeData._srcPoolId, // source pool id
 173 |      getDstPoolId(additionalArgs), //lzBridgeData._dstPoolId, // dst pool id
 174 |      refund, // refund adddress. extra gas (if any) is returned to this address
 175 |      amt2Bridge, // quantity to swap
 176 |      (amt2Bridge * (10000 - SG_FEE_BPS)) / 10000, // the min qty you would accept on the destination, fee is 6 bips
 177 |      getLzTxObj(additionalArgs), // additional gasLimit increase, airdrop, at address
 178 |      abi.encodePacked(getDestAdapter(dstChainId)),
 179 |      bridgePayload // bytes param, if you wish to send additional payload you can abi.encode() them here
 180 |  );

     |  // @audit-issue Consider named parameters
 209 |  IUTB(utb).receiveFromBridge(
 210 |      postBridge,
 211 |      target,
 212 |      paymentOperator,
 213 |      utbPayload,
 214 |      refund
 215 |  );
```
GitHub: [81-87](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L81-L87), [91-97](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L91-L97), [170-180](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L170-L180), [209-215](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L209-L215)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Consider named parameters
 129 |  IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
 130 |      .ExactInputParams({
 131 |          path: swapParams.path,
 132 |          recipient: address(this),
 133 |          amountIn: swapParams.amountIn,
 134 |          amountOutMinimum: swapParams.amountOut
 135 |      });

     |  // @audit-issue Consider named parameters
 149 |  IV3SwapRouter.ExactOutputParams memory params = IV3SwapRouter
 150 |      .ExactOutputParams({
 151 |          path: swapParams.path,
 152 |          recipient: address(this),
 153 |          //deadline: block.timestamp,
 154 |          amountOut: swapParams.amountOut,
 155 |          amountInMaximum: swapParams.amountIn
 156 |      });
```
GitHub: [129-135](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L129-L135), [149-156](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L149-L156)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Consider named parameters
 101 |  payload = abi.encode(msgType, msg.sender, _toAddress, deliverEth);

     |  // @audit-issue Consider named parameters
 103 |  payload = abi.encode(
 104 |      msgType,
 105 |      msg.sender,
 106 |      _toAddress,
 107 |      deliverEth,
 108 |      additionalPayload
 109 |  );

     |  // @audit-issue Consider named parameters
 126 |  ) = _getCallParams(
 127 |          msgType,
 128 |          _toAddress,
 129 |          _dstChainId,
 130 |          _dstGasForCall,
 131 |          deliverEth,
 132 |          payload
 133 |      );

     |  // @audit-issue Consider named parameters
 136 |  dcntEth.estimateSendAndCallFee(
 137 |      _dstChainId,
 138 |      destinationBridge,
 139 |      _amount,
 140 |      _payload,
 141 |      _dstGasForCall,
 142 |      false, // useZero
 143 |      adapterParams // Relayer adapter parameters
 144 |      // contains packet type (send and call in this case) and gasAmount
 145 |  );

     |  // @audit-issue Consider named parameters
 161 |  ) = _getCallParams(
 162 |          msgType,
 163 |          _toAddress,
 164 |          _dstChainId,
 165 |          _dstGasForCall,
 166 |          deliverEth,
 167 |          additionalPayload
 168 |      );

     |  // @audit-issue Consider named parameters
 185 |  dcntEth.sendAndCall{value: gasValue}(
 186 |      address(this), // from address that has dcntEth (so DecentRouter)
 187 |      _dstChainId,
 188 |      destinationBridge, // toAddress
 189 |      _amount, // amount
 190 |      payload, //payload (will have recipients address)
 191 |      _dstGasForCall, // dstGasForCall
 192 |      callParams // refundAddress, zroPaymentAddress, adapterParams
 193 |  );

     |  // @audit-issue Consider named parameters
 206 |  _bridgeWithPayload(
 207 |      MT_ETH_TRANSFER_WITH_PAYLOAD,
 208 |      _dstChainId,
 209 |      _toAddress,
 210 |      _amount,
 211 |      _dstGasForCall,
 212 |      additionalPayload,
 213 |      deliverEth
 214 |  );

     |  // @audit-issue Consider named parameters
 225 |  _bridgeWithPayload(
 226 |      MT_ETH_TRANSFER,
 227 |      _dstChainId,
 228 |      _toAddress,
 229 |      _amount,
 230 |      _dstGasForCall,
 231 |      bytes(""),
 232 |      deliverEth
 233 |  );

     |  // @audit-issue Consider named parameters
 257 |  emit ReceivedDecentEth(
 258 |      msgType,
 259 |      _srcChainId,
 260 |      _from,
 261 |      _to,
 262 |      _amount,
 263 |      callPayload
 264 |  );

     |  // @audit-issue Consider named parameters
 280 |  executor.execute(_from, _to, deliverEth, _amount, callPayload);
```
GitHub: [101](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L101), [103-109](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L103-L109), [126-133](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L126-L133), [136-145](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L136-L145), [161-168](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L161-L168), [185-193](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L185-L193), [206-214](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L206-L214), [225-233](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L225-L233), [257-264](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L257-L264), [280](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L280)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue Consider named parameters
  78 |  _executeWeth(from, target, amount, callPayload);

     |  // @audit-issue Consider named parameters
  80 |  _executeEth(from, target, amount, callPayload);
```
GitHub: [78](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L78), [80](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L80)


## [N-08] Duplicate strings defined

For better maintainability, consider creating and using a constant for those strings instead of duplicating it.

*Instances: 1*

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-info Duplicates of this string were detected
  93 |  string.concat("dst chain address not set ")

File: src/bridge_adapters/StargateBridgeAdapter.sol
     |  // @audit-issue First seen at src/bridge_adapters/DecentBridgeAdapter.sol:93
 159 |  string.concat("dst chain address not set ")
```
GitHub: [159](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L159)


## [N-09] Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, for readability

Well-organized data structures make code reviews easier, which may lead to fewer bugs. Consider combining related mappings into mappings to structs, so it's clear what data is related.

*Instances: 2*

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Consider whether multiple mappings can be merged into a single struct mapping
  25 |  mapping(uint256 => address) public destinationBridgeAdapter;

     |  // @audit-issue Consider whether multiple mappings can be merged into a single struct mapping
  26 |  mapping(uint256 => uint16) lzIdLookup;
```
GitHub: [25](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L25), [26](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L26)


## [N-10] NatSpec: public state variable declarations should have @notice tag

As per [Solidity NatSpec](https://docs.soliditylang.org/en/latest/natspec-format.html#tags).

*Instances: 25*

```solidity
File: src/UTB.sol

     |  // @audit-issue Add NatSpec documentation
  21 |  mapping(uint8 => address) public swappers;

     |  // @audit-issue Add NatSpec documentation
  22 |  mapping(uint8 => address) public bridgeAdapters;
```
GitHub: [21](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L21), [22](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L22)


```solidity
File: src/bridge_adapters/BaseAdapter.sol

     |  // @audit-issue Add NatSpec documentation
   7 |  address public bridgeExecutor;
```
GitHub: [7](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L7)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue Add NatSpec documentation
  13 |  uint8 public constant BRIDGE_ID = 0;

     |  // @audit-issue Add NatSpec documentation
  14 |  mapping(uint256 => address) public destinationBridgeAdapter;

     |  // @audit-issue Add NatSpec documentation
  15 |  IDecentEthRouter public router;
```
GitHub: [13](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L13), [14](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L14), [15](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L15)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Add NatSpec documentation
  22 |  uint8 public constant BRIDGE_ID = 1;

     |  // @audit-issue Add NatSpec documentation
  23 |  uint8 public constant SG_FEE_BPS = 6;

     |  // @audit-issue Add NatSpec documentation
  24 |  address public stargateEth;

     |  // @audit-issue Add NatSpec documentation
  25 |  mapping(uint256 => address) public destinationBridgeAdapter;

     |  // @audit-issue Add NatSpec documentation
  31 |  IStargateRouter public router;
```
GitHub: [22](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L22), [23](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L23), [24](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L24), [25](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L25), [31](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L31)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Add NatSpec documentation
  16 |  uint8 public constant SWAPPER_ID = 0;

     |  // @audit-issue Add NatSpec documentation
  17 |  address public uniswap_router;

     |  // @audit-issue Add NatSpec documentation
  18 |  address payable public wrapped;
```
GitHub: [16](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L16), [17](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L17), [18](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L18)


```solidity
File: lib/decent-bridge/src/DcntEth.sol

     |  // @audit-issue Add NatSpec documentation
   6 |  address public router;
```
GitHub: [6](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L6)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Add NatSpec documentation
  13 |  IWETH public weth;

     |  // @audit-issue Add NatSpec documentation
  14 |  IDcntEth public dcntEth;

     |  // @audit-issue Add NatSpec documentation
  15 |  IDecentBridgeExecutor public executor;

     |  // @audit-issue Add NatSpec documentation
  17 |  uint8 public constant MT_ETH_TRANSFER = 0;

     |  // @audit-issue Add NatSpec documentation
  18 |  uint8 public constant MT_ETH_TRANSFER_WITH_PAYLOAD = 1;

     |  // @audit-issue Add NatSpec documentation
  20 |  uint16 public constant PT_SEND_AND_CALL = 1;

     |  // @audit-issue Add NatSpec documentation
  22 |  bool public gasCurrencyIsEth; // for chains that use ETH as gas currency

     |  // @audit-issue Add NatSpec documentation
  24 |  mapping(uint16 => address) public destinationBridges;

     |  // @audit-issue Add NatSpec documentation
  25 |  mapping(address => uint256) public balanceOf;
```
GitHub: [13](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L13), [14](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L14), [15](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L15), [17](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L17), [18](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L18), [20](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L20), [22](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L22), [24](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L24), [25](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L25)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue Add NatSpec documentation
  10 |  bool public gasCurrencyIsEth; // for chains that use ETH as gas currency
```
GitHub: [10](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L10)


## [N-11] NatSpec: state variable declarations should have @dev tag

As per [Solidity NatSpec](https://docs.soliditylang.org/en/latest/natspec-format.html#tags). Note: public and non-public state variables should have a @dev tag. Only public state variables should have a @notice tag.

*Instances: 39*

```solidity
File: src/UTB.sol

     |  // @audit-issue Add NatSpec documentation
  18 |  IUTBExecutor executor;

     |  // @audit-issue Add NatSpec documentation
  19 |  IUTBFeeCollector feeCollector;

     |  // @audit-issue Add NatSpec documentation
  20 |  IWETH wrapped;

     |  // @audit-issue Add NatSpec documentation
  21 |  mapping(uint8 => address) public swappers;

     |  // @audit-issue Add NatSpec documentation
  22 |  mapping(uint8 => address) public bridgeAdapters;
```
GitHub: [18](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L18), [19](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L19), [20](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L20), [21](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L21), [22](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L22)


```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-issue Add NatSpec documentation
   9 |  address signer;

     |  // @audit-issue Add NatSpec documentation
  10 |  string constant BANNER = "\x19Ethereum Signed Message:\n32";
```
GitHub: [9](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L9), [10](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L10)


```solidity
File: src/bridge_adapters/BaseAdapter.sol

     |  // @audit-issue Add NatSpec documentation
   7 |  address public bridgeExecutor;
```
GitHub: [7](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L7)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue Add NatSpec documentation
  13 |  uint8 public constant BRIDGE_ID = 0;

     |  // @audit-issue Add NatSpec documentation
  14 |  mapping(uint256 => address) public destinationBridgeAdapter;

     |  // @audit-issue Add NatSpec documentation
  15 |  IDecentEthRouter public router;

     |  // @audit-issue Add NatSpec documentation
  16 |  mapping(uint256 => uint16) lzIdLookup;

     |  // @audit-issue Add NatSpec documentation
  17 |  mapping(uint16 => uint256) chainIdLookup;

     |  // @audit-issue Add NatSpec documentation
  18 |  bool gasIsEth;

     |  // @audit-issue Add NatSpec documentation
  19 |  address bridgeToken;
```
GitHub: [13](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L13), [14](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L14), [15](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L15), [16](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L16), [17](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L17), [18](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L18), [19](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L19)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Add NatSpec documentation
  22 |  uint8 public constant BRIDGE_ID = 1;

     |  // @audit-issue Add NatSpec documentation
  23 |  uint8 public constant SG_FEE_BPS = 6;

     |  // @audit-issue Add NatSpec documentation
  24 |  address public stargateEth;

     |  // @audit-issue Add NatSpec documentation
  25 |  mapping(uint256 => address) public destinationBridgeAdapter;

     |  // @audit-issue Add NatSpec documentation
  26 |  mapping(uint256 => uint16) lzIdLookup;

     |  // @audit-issue Add NatSpec documentation
  27 |  mapping(uint16 => uint256) chainIdLookup;

     |  // @audit-issue Add NatSpec documentation
  31 |  IStargateRouter public router;
```
GitHub: [22](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L22), [23](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L23), [24](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L24), [25](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L25), [26](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L26), [27](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L27), [31](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L31)


```solidity
File: src/swappers/SwapParams.sol

     |  // @audit-issue Add NatSpec documentation
   5 |  uint8 constant EXACT_IN = 0;

     |  // @audit-issue Add NatSpec documentation
   6 |  uint8 constant EXACT_OUT = 1;
```
GitHub: [5](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L5), [6](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L6)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Add NatSpec documentation
  16 |  uint8 public constant SWAPPER_ID = 0;

     |  // @audit-issue Add NatSpec documentation
  17 |  address public uniswap_router;

     |  // @audit-issue Add NatSpec documentation
  18 |  address payable public wrapped;
```
GitHub: [16](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L16), [17](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L17), [18](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L18)


```solidity
File: lib/decent-bridge/src/DcntEth.sol

     |  // @audit-issue Add NatSpec documentation
   6 |  address public router;
```
GitHub: [6](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L6)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Add NatSpec documentation
  13 |  IWETH public weth;

     |  // @audit-issue Add NatSpec documentation
  14 |  IDcntEth public dcntEth;

     |  // @audit-issue Add NatSpec documentation
  15 |  IDecentBridgeExecutor public executor;

     |  // @audit-issue Add NatSpec documentation
  17 |  uint8 public constant MT_ETH_TRANSFER = 0;

     |  // @audit-issue Add NatSpec documentation
  18 |  uint8 public constant MT_ETH_TRANSFER_WITH_PAYLOAD = 1;

     |  // @audit-issue Add NatSpec documentation
  20 |  uint16 public constant PT_SEND_AND_CALL = 1;

     |  // @audit-issue Add NatSpec documentation
  22 |  bool public gasCurrencyIsEth; // for chains that use ETH as gas currency

     |  // @audit-issue Add NatSpec documentation
  24 |  mapping(uint16 => address) public destinationBridges;

     |  // @audit-issue Add NatSpec documentation
  25 |  mapping(address => uint256) public balanceOf;
```
GitHub: [13](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L13), [14](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L14), [15](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L15), [17](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L17), [18](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L18), [20](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L20), [22](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L22), [24](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L24), [25](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L25)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue Add NatSpec documentation
   9 |  IWETH weth;

     |  // @audit-issue Add NatSpec documentation
  10 |  bool public gasCurrencyIsEth; // for chains that use ETH as gas currency
```
GitHub: [9](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L9), [10](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L10)


## [N-12] Redundant `else` block

One level of nesting can be removed by not having an else block when the if-block returns.

*Instances: 2*

```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue The else clause is unnecessary
  76 |  if (remainingBalance == 0) {
  77 |      return;
  78 |  }
```
GitHub: [76-78](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L76-L78)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue The else clause is unnecessary
  68 |  if (swapParams.path.length == 0) {
  69 |      return swapNoPath(swapParams, receiver, refund);
  70 |  }
```
GitHub: [68-70](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L68-L70)


## [N-13] State variables should include comments

Consider adding some comments on critical state variables to explain what they are supposed to do: this will help for future code reviews.

*Instances: 39*

```solidity
File: src/UTB.sol

     |  // @audit-issue Consider adding comments explaining this state var
  18 |  IUTBExecutor executor;

     |  // @audit-issue Consider adding comments explaining this state var
  19 |  IUTBFeeCollector feeCollector;

     |  // @audit-issue Consider adding comments explaining this state var
  20 |  IWETH wrapped;

     |  // @audit-issue Consider adding comments explaining this state var
  21 |  mapping(uint8 => address) public swappers;

     |  // @audit-issue Consider adding comments explaining this state var
  22 |  mapping(uint8 => address) public bridgeAdapters;
```
GitHub: [18](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L18), [19](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L19), [20](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L20), [21](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L21), [22](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L22)


```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-issue Consider adding comments explaining this state var
   9 |  address signer;

     |  // @audit-issue Consider adding comments explaining this state var
  10 |  string constant BANNER = "\x19Ethereum Signed Message:\n32";
```
GitHub: [9](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L9), [10](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L10)


```solidity
File: src/bridge_adapters/BaseAdapter.sol

     |  // @audit-issue Consider adding comments explaining this state var
   7 |  address public bridgeExecutor;
```
GitHub: [7](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L7)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue Consider adding comments explaining this state var
  13 |  uint8 public constant BRIDGE_ID = 0;

     |  // @audit-issue Consider adding comments explaining this state var
  14 |  mapping(uint256 => address) public destinationBridgeAdapter;

     |  // @audit-issue Consider adding comments explaining this state var
  15 |  IDecentEthRouter public router;

     |  // @audit-issue Consider adding comments explaining this state var
  16 |  mapping(uint256 => uint16) lzIdLookup;

     |  // @audit-issue Consider adding comments explaining this state var
  17 |  mapping(uint16 => uint256) chainIdLookup;

     |  // @audit-issue Consider adding comments explaining this state var
  18 |  bool gasIsEth;

     |  // @audit-issue Consider adding comments explaining this state var
  19 |  address bridgeToken;
```
GitHub: [13](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L13), [14](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L14), [15](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L15), [16](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L16), [17](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L17), [18](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L18), [19](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L19)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Consider adding comments explaining this state var
  22 |  uint8 public constant BRIDGE_ID = 1;

     |  // @audit-issue Consider adding comments explaining this state var
  23 |  uint8 public constant SG_FEE_BPS = 6;

     |  // @audit-issue Consider adding comments explaining this state var
  24 |  address public stargateEth;

     |  // @audit-issue Consider adding comments explaining this state var
  25 |  mapping(uint256 => address) public destinationBridgeAdapter;

     |  // @audit-issue Consider adding comments explaining this state var
  26 |  mapping(uint256 => uint16) lzIdLookup;

     |  // @audit-issue Consider adding comments explaining this state var
  27 |  mapping(uint16 => uint256) chainIdLookup;

     |  // @audit-issue Consider adding comments explaining this state var
  31 |  IStargateRouter public router;
```
GitHub: [22](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L22), [23](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L23), [24](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L24), [25](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L25), [26](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L26), [27](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L27), [31](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L31)


```solidity
File: src/swappers/SwapParams.sol

     |  // @audit-issue Consider adding comments explaining this state var
   5 |  uint8 constant EXACT_IN = 0;

     |  // @audit-issue Consider adding comments explaining this state var
   6 |  uint8 constant EXACT_OUT = 1;
```
GitHub: [5](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L5), [6](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L6)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Consider adding comments explaining this state var
  16 |  uint8 public constant SWAPPER_ID = 0;

     |  // @audit-issue Consider adding comments explaining this state var
  17 |  address public uniswap_router;

     |  // @audit-issue Consider adding comments explaining this state var
  18 |  address payable public wrapped;
```
GitHub: [16](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L16), [17](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L17), [18](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L18)


```solidity
File: lib/decent-bridge/src/DcntEth.sol

     |  // @audit-issue Consider adding comments explaining this state var
   6 |  address public router;
```
GitHub: [6](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L6)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Consider adding comments explaining this state var
  13 |  IWETH public weth;

     |  // @audit-issue Consider adding comments explaining this state var
  14 |  IDcntEth public dcntEth;

     |  // @audit-issue Consider adding comments explaining this state var
  15 |  IDecentBridgeExecutor public executor;

     |  // @audit-issue Consider adding comments explaining this state var
  17 |  uint8 public constant MT_ETH_TRANSFER = 0;

     |  // @audit-issue Consider adding comments explaining this state var
  18 |  uint8 public constant MT_ETH_TRANSFER_WITH_PAYLOAD = 1;

     |  // @audit-issue Consider adding comments explaining this state var
  20 |  uint16 public constant PT_SEND_AND_CALL = 1;

     |  // @audit-issue Consider adding comments explaining this state var
  22 |  bool public gasCurrencyIsEth; // for chains that use ETH as gas currency

     |  // @audit-issue Consider adding comments explaining this state var
  24 |  mapping(uint16 => address) public destinationBridges;

     |  // @audit-issue Consider adding comments explaining this state var
  25 |  mapping(address => uint256) public balanceOf;
```
GitHub: [13](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L13), [14](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L14), [15](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L15), [17](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L17), [18](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L18), [20](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L20), [22](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L22), [24](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L24), [25](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L25)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue Consider adding comments explaining this state var
   9 |  IWETH weth;

     |  // @audit-issue Consider adding comments explaining this state var
  10 |  bool public gasCurrencyIsEth; // for chains that use ETH as gas currency
```
GitHub: [9](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L9), [10](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L10)


## [N-14] Syntax: variables need not be initialized to zero

The default value for variables is zero, so initializing them to zero is superfluous. This will also save gas if the variable is a mutable state variable.

*Instances: 1*

```solidity
File: src/UTB.sol

     |  // @audit-issue Initial value is unnecessary
 234 |  uint value = 0;
```
GitHub: [234](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L234)


## [N-15] Typos

Consider correcting these spelling errors.

*Instances: 1*

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Typo: adddress
 174 |  refund, // refund adddress. extra gas (if any) is returned to this address
```
GitHub: [174](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L174)


## [N-16] Unnecessary cast

Unnecessary casts can be removed.

*Instances: 1*

```solidity
File: src/bridge_adapters/BaseAdapter.sol

     |  // @audit-issue Cast to address is redundant
  13 |  msg.sender == address(bridgeExecutor),
```
GitHub: [13](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L13)


## [N-17] `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`

Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739). If all arguments are strings and or bytes, `bytes.concat()` should be used instead

*Instances: 1*

```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-issue Use abi.encode instead to prevent dynamic-length hash collisions
  49 |  bytes32 constructedHash = keccak256(
  50 |      abi.encodePacked(BANNER, keccak256(packedInfo))
  51 |  );
```
GitHub: [49-51](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L49-L51)


## [N-18] `require()`/`revert()` statements should have descriptive reason strings

The error message should clearly state _how_/_why_ something is an error, not just that a specific error state was reached; someone shouldn't have to read the code in order to see how to resolve the problem.

*Instances: 1*

```solidity
File: lib/decent-bridge/src/DcntEth.sol

     |  // @audit-issue Add an error description, ideally with a custom error
   9 |  require(msg.sender == router);
```
GitHub: [9](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L9)
