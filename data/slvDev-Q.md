## Low Findings

|    | Issue | Instances |
|----|-------|:---------:|
| [L-01] | Potential Gas Griefing Due to Non-Handling of Return Data in External Calls | 2 |
| [L-02] | `.call` bypasses function existence check, type checking and argument packing | 5 |
| [L-03] | Unbounded Gas Consumption on External Calls | 5 |
| [L-04] | Use of `ecrecover` is susceptible to signature malleability | 1 |
| [L-05] | Upgradable contracts not taken into account | 20 |
| [L-06] | Risk of Permanently Locked Ether | 12 |
| [L-07] | Missing Contract-Existence Checks Before Low-Level Calls | 7 |
| [L-08] | Loss of precision | 2 |
| [L-09] | Missing `address(0)` Check in Constructor | 3 |


## NonCritical Findings

|    | Issue | Instances |
|----|-------|:---------:|
| [N-01] | Variables need not be initialized to zero | 5 |
| [N-02] | `require()/revert()` statements without reason strings | 1 |
| [N-03] | Consider using descriptive `constant`s when passing zero as a function argument | 1 |
| [N-04] | Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, for readability | 6 |
| [N-05] | Remove Unused `private` Functions | 1 |
| [N-06] | High cyclomatic complexity | 2 |
| [N-07] | State-Altering Functions Should Emit Events | 12 |
| [N-08] | Contract/Libraries Names Do Not Match Their Filenames | 1 |
| [N-09] | Function/Constructor Argument Names Not in mixedCase | 26 |
| [N-10] | Contract/Library Names Not in `CapWords` Style (CamelCase) | 1 |
| [N-11] | Leverage Recent Solidity Features with `0.8.23` | 11 |


## Low Findings Details


### [L-01] Potential Gas Griefing Due to Non-Handling of Return Data in External Calls

Due to the EVM architecture, return data (bool success,) has to be stored.
However, when 'out' and 'outsize' values are given (0,0), this storage disappears.
This can lead to potential gas griefing/theft issues, especially when dealing with external contracts.
```solidity
assembly {
    success: = call(gas(), dest, amount, 0, 0)
}
require(success, "transfer failed");

```
Consider using a safe call pattern above to avoid these issues.
The following instances show the unsafe external call patterns found in the code.

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

33: (bool success, ) = target.call(callPayload);
61: (bool success, ) = target.call{value: amount}(callPayload);
```
[33](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L33) | [61](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L61)
</details>


### [L-02] `.call` bypasses function existence check, type checking and argument packing

Using the `.call` method in Solidity enables direct communication with an address, bypassing function existence checks, type checking, and argument packing.
While this can save gas and provide flexibility, it can also introduce security risks and potential errors. The absence of these checks can lead to unexpected behavior if the callee contract's interface changes or if the input parameters are not crafted with care.
The resolution to these issues is to use Solidity's high-level interface for calling functions when possible, as it automatically manages these aspects.
If using `.call` is necessary, ensure that the inputs are carefully validated and that awareness of the called contract's behavior is maintained.

<details>
<summary><i>5 issue instances in 2 files:</i></summary>

```solidity
File: src/UTBExecutor.sol

52: target.call{value: amount}(payload)
65: target.call{value: extraNative}(payload)
70: target.call(payload)
```
[52](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L52) | [65](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L65) | [70](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L70)

```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

33: target.call(callPayload)
61: target.call{value: amount}(callPayload)
```
[33](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L33) | [61](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L61)
</details>


### [L-03] Unbounded Gas Consumption on External Calls

External calls in your code don't specify a gas limit, which can lead to scenarios where the recipient consumes all transaction's gas causing it to revert. 
Consider using `addr.call{gas: <amount>}("")` to set a gas limit and prevent potential reversion due to gas consumption.

<details>
<summary><i>5 issue instances in 2 files:</i></summary>

```solidity
File: src/UTBExecutor.sol

52: (success, ) = target.call{value: amount}(payload);
65: (success, ) = target.call{value: extraNative}(payload);
70: (success, ) = target.call(payload);
```
[52](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L52) | [65](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L65) | [70](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L70)

```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

33: (bool success, ) = target.call(callPayload);
61: (bool success, ) = target.call{value: amount}(callPayload);
```
[33](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L33) | [61](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L61)
</details>


### [L-04] Use of `ecrecover` is susceptible to signature malleability

The built-in EVM precompile ecrecover is susceptible to signature malleability, which could lead to replay attacks.
References: https://swcregistry.io/docs/SWC-117, https://swcregistry.io/docs/SWC-121.
While this is not immediately exploitable, this may become a vulnerability if used elsewhere.
[Consider using OpenZeppelin’s ECDSA library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol) (which prevents this malleability) instead of the built-in function.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/UTBFeeCollector.sol

53: ecrecover(constructedHash, v, r, s)
```
[53](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L53)
</details>


### [L-05] Upgradable contracts not taken into account

In the realm of blockchain development, it's crucial to consider the impact of upgradable contracts, especially when handling token addresses through interfaces like IERC20.
These contracts can evolve over time, potentially altering their behavior or interface. Such changes may lead to compatibility issues or security vulnerabilities in the protocol that relies on them.

<details>
<summary><i>20 issue instances in 6 files:</i></summary>

```solidity
File: src/UTB.sol

83: IERC20(swapParams.tokenIn).transferFrom(
90: IERC20(swapParams.tokenIn).approve(
152: IERC20(tokenOut).approve(address(executor), amountOut);
236: IERC20(fees.feeToken).transferFrom(
241: IERC20(fees.feeToken).approve(
```
[83](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L83) | [90](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L90) | [152](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L152) | [236](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L236) | [241](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L241)

```solidity
File: src/UTBExecutor.sol

61: IERC20(token).transferFrom(msg.sender, address(this), amount);
62: IERC20(token).approve(paymentOperator, amount);
80: IERC20(token).transfer(refund, remainingBalance);
```
[61](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L61) | [62](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L62) | [80](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L80)

```solidity
File: src/UTBFeeCollector.sol

56: IERC20(fees.feeToken).transferFrom(
73: IERC20(token).transfer(owner, amount);
```
[56](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L56) | [73](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L73)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

139: IERC20(swapParams.tokenIn).transferFrom(
145: IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn);
```
[109](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L109) | [114](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L114) | [139](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L139) | [145](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L145)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

207: IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn);
```
[207](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L207)

```solidity
File: src/swappers/UniSwapper.sol

44: IERC20(token).transfer(user, amount);
55: IERC20(token).transfer(recipient, amount);
83: IERC20(swapParams.tokenIn).transferFrom(
137: IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
158: IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
```
[44](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L44) | [55](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L55) | [83](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L83) | [137](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L137) | [158](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L158)
</details>


### [L-06] Risk of Permanently Locked Ether

When Ether is mistakenly sent to a contract without a means of retrieval, it becomes irrevocably locked.
Incidents of accidental Ether transfers have been observed even in high-profile projects, potentially leading to significant financial setbacks.
To enhance contract resilience, it's recommended to incorporate an "Ether recovery" function to serve as a protective measure against unintended Ether lockups.

<details>
<summary><i>12 issue instances in 6 files:</i></summary>

```solidity
File: src/UTB.sol

339: receive() external payable {}
341: fallback() external payable {}
```
[339](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L339) | [341](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L341)

```solidity
File: src/UTBFeeCollector.sol

77: receive() external payable {}
79: fallback() external payable {}
```
[77](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L77) | [79](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L79)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

156: receive() external payable {}
158: fallback() external payable {}
```
[156](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L156) | [158](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L158)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

218: receive() external payable {}
220: fallback() external payable {}
```
[218](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L218) | [220](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L220)

```solidity
File: src/swappers/UniSwapper.sol

171: receive() external payable {}
173: fallback() external payable {}
```
[171](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L171) | [173](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L173)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

337: receive() external payable {}
339: fallback() external payable {}
```
[337](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L337) | [339](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L339)
</details>


### [L-07] Missing Contract-Existence Checks Before Low-Level Calls

When making low-level calls, it's crucial to ensure the existence of the contract at the specified address. 
If the contract doesn't exist at the given address, low-level calls will still return success, potentially causing errors in the code execution.
Therefore, alongside zero-address checks, adding an additional check to verify that <address>.code.length > 0 before making low-level calls would be recommended.

<details>
<summary><i>7 issue instances in 2 files:</i></summary>

```solidity
File: src/UTBExecutor.sol

52: (success, ) = target.call{value: amount}(payload);
54: (refund.call{value: amount}(""));
65: (success, ) = target.call{value: extraNative}(payload);
67: (refund.call{value: extraNative}(""));
70: (success, ) = target.call(payload);
```
[52](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L52) | [54](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L54) | [65](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L65) | [67](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L67) | [70](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L70)

```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

33: (bool success, ) = target.call(callPayload);
61: (bool success, ) = target.call{value: amount}(callPayload);
```
[33](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L33) | [61](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L61)
</details>


### [L-08] Loss of precision

Division by large numbers may result in the result being zero, due to Solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator.

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

66: return (amt2Bridge * (1e4 - SG_FEE_BPS)) / 1e4;
176: (amt2Bridge * (10000 - SG_FEE_BPS)) / 10000, // the min qty you would accept on the destination, fee is 6 bips
```
[66](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L66) | [176](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L176)
</details>


### [L-09] Missing `address(0)` Check in Constructor

The constructor does not include a check for `address(0)` when set state variables that hold addresses.
Initializing a state variable with `address(0)` can lead to unintended behavior and vulnerabilities in the contract, such as sending funds to an inaccessible address. 
It is recommended to include a validation step to ensure that address parameters are not set to `address(0)`.

<details>
<summary><i>3 issue instances in 3 files:</i></summary>

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

/// @audit `_bridgeToken` has lack of `address(0)` check before use
20: constructor(bool _gasIsEth, address _bridgeToken) BaseAdapter() {
```
[20](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L20)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

/// @audit `_wethAddress` has lack of `address(0)` check before use
/// @audit `_executor` has lack of `address(0)` check before use
26: constructor(
        address payable _wethAddress,
        bool gasIsEth,
        address _executor
    ) Owned(msg.sender) {
```
[26](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L26)

```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

/// @audit `_weth` has lack of `address(0)` check before use
11: constructor(address _weth, bool gasIsEth) Owned(msg.sender) {
```
[11](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L11)
</details>


### [N-01] Variables need not be initialized to zero

By default, `int/uint` variables in Solidity are initialized to `zero`.
Explicitly setting variables to zero during their declaration is redundant and might cause confusion.
Removing the explicit zero initialization can improve code readability and understanding.

<details>
<summary><i>5 issue instances in 5 files:</i></summary>

```solidity
File: src/UTB.sol

234: uint value = 0;
```
[234](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L234)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

13: uint8 public constant BRIDGE_ID = 0;
```
[13](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L13)

```solidity
File: src/swappers/SwapParams.sol

5: uint8 constant EXACT_IN = 0;
```
[5](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L5)

```solidity
File: src/swappers/UniSwapper.sol

16: uint8 public constant SWAPPER_ID = 0;
```
[16](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L16)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

17: uint8 public constant MT_ETH_TRANSFER = 0;
```
[17](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L17)
</details>


### [N-02] `require()/revert()` statements without reason strings

In Solidity, require() and revert() functions can include an optional 'reason' string that describes what caused the function to fail. This string can be incredibly helpful during testing and debugging, as it provides more context for what went wrong.

Some `require()/revert()` statements do not have these descriptive reason strings. For better error handling and debugging, it is strongly recommended to add descriptive reason strings to all `require()/revert()` statements.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: lib/decent-bridge/src/DcntEth.sol

9: require(msg.sender == router)
```
[9](https://github.com/decentxyz/decent-bridge/tree/main/src/DcntEth.sol#L9)
</details>


### [N-03] Consider using descriptive `constant`s when passing zero as a function argument

In instances where utilizing a zero parameter is essential, it is recommended to employ descriptive constants or an enum instead of directly integrating zero within function calls.
This strategy aids in clearly articulating the caller's intention and minimizes the risk of errors.
Emitting zero also not recomended, as it is not clear what the intention is.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/UTBExecutor.sol

28: execute(target, paymentOperator, payload, token, amount, refund, 0)
```
[28](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L28)
</details>


### [N-04] Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, for readability

Combining multiple address/ID mappings into a single mapping to a struct can enhance code clarity and maintainability. 
Consider refactoring multiple mappings into a single mapping with a struct for cleaner code structure.
This arrangement also promotes a more organized contract structure, making it easier for developers to navigate and understand.

<details>
<summary><i>6 issue instances in 3 files:</i></summary>

```solidity
File: src/UTB.sol

21: mapping(uint8 => address) public swappers
22: mapping(uint8 => address) public bridgeAdapters
```
[21](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L21) | [22](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L22)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

14: mapping(uint256 => address) public destinationBridgeAdapter
16: mapping(uint256 => uint16) lzIdLookup
```
[14](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L14) | [16](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L16)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

25: mapping(uint256 => address) public destinationBridgeAdapter
26: mapping(uint256 => uint16) lzIdLookup
```
[25](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L25) | [26](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L26)
</details>


### [N-05] Remove Unused `private` Functions

`private` functions that are not called anywhere within the same contract are redundant and should be removed to save deployment gas.
Unlike `public` and `internal` functions, `private` functions cannot be called by derived contracts or externally, they can only be called within the contract they are defined.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

100: function getValue(
        bytes calldata additionalArgs,
        uint256 amt2Bridge
    ) private view returns (uint value)
```
[100](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L100)
</details>


### [N-06] High cyclomatic complexity

Functions with high cyclomatic complexity are harder to understand, test, and maintain.
Consider breaking down these blocks into more manageable units, by splitting things into utility functions,
by reducing nesting, and by using early returns.

[Learn More About Cyclomatic Complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity)

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: src/UTBExecutor.sol

/// @audit function `execute` has a cyclomatic complexity of 6
41: function execute(
        address target,
        address paymentOperator,
        bytes memory payload,
        address token,
        uint amount,
        address payable refund,
        uint extraNative
    ) public onlyOwner {
```
[41](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L41)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

/// @audit function `onOFTReceived` has a cyclomatic complexity of 6
237: function onOFTReceived(
        uint16 _srcChainId,
        bytes calldata,
        uint64,
        bytes32,
        uint _amount,
        bytes memory _payload
    ) external override onlyLzApp {
```
[237](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L237)
</details>


### [N-07] State-Altering Functions Should Emit Events

Functions that alter state should emit events to inform users of the state change.
This is crucial for functions that modify the state and don't return a value.
The absence of events in such scenarios could lead to lack of transparency and traceability, undermining the contract's reliability.

<details>
<summary><i>12 issue instances in 8 files:</i></summary>

```solidity
File: src/UTB.sol

29: executor = IUTBExecutor(_executor);
37: wrapped = IWETH(_wrapped);
45: feeCollector = IUTBFeeCollector(_feeCollector);
```
[29](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L29) | [37](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L37) | [45](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L45)

```solidity
File: src/UTBFeeCollector.sol

19: signer = _signer;
```
[19](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L19)

```solidity
File: src/bridge_adapters/BaseAdapter.sol

20: bridgeExecutor = _executor;
```
[20](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L20)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

27: router = IDecentEthRouter(payable(_router));
```
[27](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L27)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

34: router = IStargateRouter(_router);
38: stargateEth = _sgEth;
```
[34](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L34) | [38](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L38)

```solidity
File: src/swappers/UniSwapper.sol

21: uniswap_router = _router;
25: wrapped = _wrapped;
```
[21](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L21) | [25](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L25)

```solidity
File: lib/decent-bridge/src/DcntEth.sol

21: router = _router;
```
[21](https://github.com/decentxyz/decent-bridge/tree/main/src/DcntEth.sol#L21)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

69: dcntEth = IDcntEth(_addr);
```
[69](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L69)
</details>


### [N-08] Contract/Libraries Names Do Not Match Their Filenames

According to the Solidity Style Guide, contract names should match their filenames. 
Mismatching contract names and filenames can lead to confusion and make the code harder to maintain and review.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/swappers/SwapParams.sol

3: library SwapDirection {
```
[3](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L3)
</details>


### [N-09] Function/Constructor Argument Names Not in mixedCase

Underscore before of after function argument names is a common convention in Solidity NOT a documentation requirement.

Function arguments should use mixedCase for better readability and consistency with Solidity style guidelines. 
Examples of good practice include: initialSupply, account, recipientAddress, senderAddress, newOwner. 
[More information in Documentation](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#function-argument-names)

Rule exceptions
- Allow constant variable name/symbol/decimals to be lowercase (ERC20).
- Allow `_` at the beginning of the mixedCase match for `private variables` and `unused parameters`.

<details>
<summary><i>26 issue instances in 9 files:</i></summary>

```solidity
File: src/UTB.sol

28: function setExecutor(address _executor) public onlyOwner {
36: function setWrapped(address payable _wrapped) public onlyOwner {
44: function setFeeCollector(address payable _feeCollector) public onlyOwner {
```
[28](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L28) | [36](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L36) | [44](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L44)

```solidity
File: src/UTBFeeCollector.sol

18: function setSigner(address _signer) public onlyOwner {
```
[18](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L18)

```solidity
File: src/bridge_adapters/BaseAdapter.sol

18: function setBridgeExecutor(address _executor) public onlyOwner {
```
[18](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L18)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

25: function setRouter(address _router) public onlyOwner {
20: constructor(bool _gasIsEth, address _bridgeToken) BaseAdapter() {
```
[25](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L25) | [20](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L20)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

32: function setRouter(address _router) public onlyOwner {
36: function setStargateEth(address _sgEth) public onlyOwner {
```
[32](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L32) | [36](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L36)

```solidity
File: src/swappers/UniSwapper.sol

19: function setRouter(address _router) public onlyOwner {
23: function setWrapped(address payable _wrapped) public onlyOwner {
```
[19](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L19) | [23](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L23)

```solidity
File: lib/decent-bridge/src/DcntEth.sol

20: function setRouter(address _router) public {
23: function mint(address _to, uint256 _amount) public onlyRouter {
27: function burn(address _from, uint256 _amount) public onlyRouter {
31: function mintByOwner(address _to, uint256 _amount) public onlyOwner {
35: function burnByOwner(address _from, uint256 _amount) public onlyOwner {
```
[20](https://github.com/decentxyz/decent-bridge/tree/main/src/DcntEth.sol#L20) | [23](https://github.com/decentxyz/decent-bridge/tree/main/src/DcntEth.sol#L23) | [27](https://github.com/decentxyz/decent-bridge/tree/main/src/DcntEth.sol#L27) | [31](https://github.com/decentxyz/decent-bridge/tree/main/src/DcntEth.sol#L31) | [35](https://github.com/decentxyz/decent-bridge/tree/main/src/DcntEth.sol#L35)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

68: function registerDcntEth(address _addr) public onlyOwner {
73: function addDestinationBridge(
        uint16 _dstChainId,
        address _routerAddress
    ) public onlyOwner {
79: function _getCallParams(
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
112: function estimateSendAndCallFee(
        uint8 msgType,
        uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        uint64 _dstGasForCall,
        bool deliverEth,
        bytes memory payload
    ) public view returns (uint nativeFee, uint zroFee) {
147: function _bridgeWithPayload(
        uint8 msgType,
        uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        uint64 _dstGasForCall,
        bytes memory additionalPayload,
        bool deliverEth
    ) internal {
197: function bridgeWithPayload(
        uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        bool deliverEth,
        uint64 _dstGasForCall,
        bytes memory additionalPayload
    ) public payable {
218: function bridge(
        uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        uint64 _dstGasForCall,
        bool deliverEth // if false, delivers WETH
    ) public payable {
237: function onOFTReceived(
        uint16 _srcChainId,
        bytes calldata,
        uint64,
        bytes32,
        uint _amount,
        bytes memory _payload
    ) external override onlyLzApp {
26: constructor(
        address payable _wethAddress,
        bool gasIsEth,
        address _executor
    ) Owned(msg.sender) {
```
[68](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L68) | [73](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L73) | [79](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L79) | [112](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L112) | [147](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L147) | [197](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L197) | [218](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L218) | [237](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L237) | [26](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L26)

```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

11: constructor(address _weth, bool gasIsEth) Owned(msg.sender) {
```
[11](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L11)
</details>


### [N-10] Contract/Library Names Not in `CapWords` Style (CamelCase)

Contracts and libraries should be named using the `CapWords` style for better readability and consistency with Solidity style guidelines.
Examples of good practice include: SimpleToken, SmartBank, CertificateHashRepository, Player, Congress, Owned.
[More information in Documentation](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#contract-and-library-names)

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/UTB.sol

13: contract UTB is Owned {
```
[13](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L13)
</details>


### [N-11] Leverage Recent Solidity Features with `0.8.23`

The recent updates in Solidity provide several features and optimizations that, when leveraged appropriately, can significantly improve your contract's code clarity and maintainability.
Key enhancements include the use of push0 for placing 0 on the stack for EVM versions starting from "Shanghai", making your code simpler and more straightforward.
Moreover, Solidity has extended NatSpec documentation support to enum and struct definitions, facilitating more comprehensive and insightful code documentation.

Additionally, the re-implementation of the UnusedAssignEliminator and UnusedStoreEliminator in the Solidity optimizer provides the ability to remove unused assignments in deeply nested loops.
This results in a cleaner, more efficient contract code, reducing clutter and potential points of confusion during code review or debugging.
It's recommended to make full use of these features and optimizations to enhance the robustness and readability of your smart contracts.

<details>
<summary><i>11 issue instances in 11 files:</i></summary>

```solidity
File: src/UTB.sol

2: pragma solidity ^0.8.0;
```

| [Line #2](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L2) | 
```solidity
File: src/UTBExecutor.sol

2: pragma solidity ^0.8.0;
```

| [Line #2](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L2) | 
```solidity
File: src/UTBFeeCollector.sol

2: pragma solidity ^0.8.0;
```

| [Line #2](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L2) | 
```solidity
File: src/bridge_adapters/BaseAdapter.sol

2: pragma solidity ^0.8.0;
```

| [Line #2](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L2) | 
```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

2: pragma solidity ^0.8.0;
```

| [Line #2](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L2) | 
```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

2: pragma solidity ^0.8.0;
```

| [Line #2](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L2) | 
```solidity
File: src/swappers/SwapParams.sol

2: pragma solidity ^0.8.0;
```

| [Line #2](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L2) | 
```solidity
File: src/swappers/UniSwapper.sol

2: pragma solidity ^0.8.0;
```

| [Line #2](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L2) | 
```solidity
File: lib/decent-bridge/src/DcntEth.sol

2: pragma solidity ^0.8.13;
```

| [Line #2](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L2) | 
```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

2: pragma solidity ^0.8.13;
```

| [Line #2](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L2) | 
```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

2: pragma solidity ^0.8.0;
```

| [Line #2](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L2) | </details>

