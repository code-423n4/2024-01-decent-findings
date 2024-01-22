> Please note: this report has been reviewed to ensure it *does not* contain automated findings from the bot race winner or 4naly3er

# Summary

## Gas Optimizations
| # | Title | Instances |
| - | - | - |
| G-01 | [Assigning to structs can be more efficient](#g-01-assigning-to-structs-can-be-more-efficient) | 3 |
| G-02 | [Avoid zero transfer to save gas](#g-02-avoid-zero-transfer-to-save-gas) | 23 |
| G-03 | [Cache multiple accesses of mapping/array values](#g-03-cache-multiple-accesses-of-mappingarray-values) | 2 |
| G-04 | [Combine multiple mappings with the same key type where appropriate to reduce gas usage](#g-04-combine-multiple-mappings-with-the-same-key-type-where-appropriate-to-reduce-gas-usage) | 2 |
| G-05 | [Consider changing some `public` variables to `private`/`internal`](#g-05-consider-changing-some-public-variables-to-privateinternal) | 25 |
| G-06 | [Function result should be cached](#g-06-function-result-should-be-cached) | 4 |
| G-07 | [Inline `modifier`s that are only used once, to save gas](#g-07-inline-modifiers-that-are-only-used-once-to-save-gas) | 9 |
| G-08 | [Mutable state variables explicitly initialized to zero consume gas unnecessarily](#g-08-mutable-state-variables-explicitly-initialized-to-zero-consume-gas-unnecessarily) | 1 |
| G-09 | [Not using the named return variable is confusing and can waste gas](#g-09-not-using-the-named-return-variable-is-confusing-and-can-waste-gas) | 10 |
| G-10 | [Public functions not used internally can be marked as external to save gas](#g-10-public-functions-not-used-internally-can-be-marked-as-external-to-save-gas) | 44 |
| G-11 | [Reduce deployment gas costs by fine-tuning IPFS file hashes](#g-11-reduce-deployment-gas-costs-by-fine-tuning-ipfs-file-hashes) | 11 |
| G-12 | [Redundant type conversion](#g-12-redundant-type-conversion) | 1 |
| G-13 | [Same cast is done multiple times](#g-13-same-cast-is-done-multiple-times) | 13 |
| G-14 | [Simple checks for zero can be done using assembly to save gas](#g-14-simple-checks-for-zero-can-be-done-using-assembly-to-save-gas) | 16 |
| G-15 | [Subtraction can potentially be marked `unchecked` to save gas](#g-15-subtraction-can-potentially-be-marked-unchecked-to-save-gas) | 8 |
| G-16 | [The result of function calls should be cached rather than re-calling the function](#g-16-the-result-of-function-calls-should-be-cached-rather-than-re-calling-the-function) | 4 |
| G-17 | [Use != 0 instead of > 0](#g-17-use--0-instead-of--0) | 1 |
| G-18 | [Use Solady library where possible to save gas](#g-18-use-solady-library-where-possible-to-save-gas) | 2 |
| G-19 | [Use assembly to calculate hashes](#g-19-use-assembly-to-calculate-hashes) | 2 |
| G-20 | [Use assembly to perform external calls, in order to save gas](#g-20-use-assembly-to-perform-external-calls-in-order-to-save-gas) | 51 |
| G-21 | [Use hardcoded address instead of `address(this)`](#g-21-use-hardcoded-address-instead-of-addressthis) | 26 |
| G-22 | [Use named return values](#g-22-use-named-return-values) | 14 |
| G-23 | [Use of low-level `call()` can be optimized with assembly](#g-23-use-of-low-level-call-can-be-optimized-with-assembly) | 7 |
| G-24 | [Using `storage` instead of `memory` for structs/arrays saves gas](#g-24-using-storage-instead-of-memory-for-structsarrays-saves-gas) | 4 |
| G-25 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#g-25-x--y-costs-more-gas-than-x--x--y-for-state-variables) | 2 |
| G-26 | [`abi.encode()` may be less efficient than `abi.encodePacked()`](#g-26-abiencode-may-be-less-efficient-than-abiencodepacked) | 7 |

# Gas Optimizations

## [G-01] Assigning to structs can be more efficient

By changing the pattern of assigning value to the structure, gas savings of ~130 per instance are achieved. In addition, this use will provide significant savings in distribution costs.  

Instead of:

```solidity     
MyStruct memory myStruct = MyStruct(_a, _b, _c); 
```

write  

```solidity     
MyStruct memory myStruct;     
myStruct.a = _a;     
myStruct.b = _b;     
myStruct.c = _c; 
```


*Instances: 3*

```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Assign struct fields individually to save gas
 129 |  IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
 130 |      .ExactInputParams({
 131 |          path: swapParams.path,
 132 |          recipient: address(this),
 133 |          amountIn: swapParams.amountIn,
 134 |          amountOutMinimum: swapParams.amountOut
 135 |      });

     |  // @audit-issue Assign struct fields individually to save gas
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

     |  // @audit-issue Assign struct fields individually to save gas
 170 |  ICommonOFT.LzCallParams memory callParams = ICommonOFT.LzCallParams({
 171 |      refundAddress: payable(msg.sender),
 172 |      zroPaymentAddress: address(0x0),
 173 |      adapterParams: adapterParams
 174 |  });
```
GitHub: [170-174](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L170-L174)


## [G-02] Avoid zero transfer to save gas

In Solidity, unnecessary operations can waste gas. For example, a transfer function without a zero amount check uses gas even if called with a zero amount, since the contract state remains unchanged. Implementing a zero amount check avoids these unnecessary function calls, saving gas and improving efficiency.

*Instances: 23*

```solidity
File: src/UTB.sol

     |  // @audit-issue Check transfer amount > 0
  83 |  IERC20(swapParams.tokenIn).transferFrom(
  84 |      msg.sender,
  85 |      address(this),
  86 |      swapParams.amountIn
  87 |  );

     |  // @audit-issue Check transfer amount > 0
 236 |  IERC20(fees.feeToken).transferFrom(
 237 |      msg.sender,
 238 |      address(this),
 239 |      fees.feeAmount
 240 |  );
```
GitHub: [83-87](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L83-L87), [236-240](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L236-L240)


```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Check transfer amount > 0
  61 |  IERC20(token).transferFrom(msg.sender, address(this), amount);

     |  // @audit-issue Check transfer amount > 0
  80 |  IERC20(token).transfer(refund, remainingBalance);
```
GitHub: [61](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L61), [80](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L80)


```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-issue Check transfer amount > 0
  56 |  IERC20(fees.feeToken).transferFrom(
  57 |      utb,
  58 |      address(this),
  59 |      fees.feeAmount
  60 |  );

     |  // @audit-issue Check transfer amount > 0
  73 |  IERC20(token).transfer(owner, amount);
```
GitHub: [56-60](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L56-L60), [73](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L73)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue Check transfer amount > 0
 109 |  IERC20(bridgeToken).transferFrom(
 110 |      msg.sender,
 111 |      address(this),
 112 |      amt2Bridge
 113 |  );

     |  // @audit-issue Check transfer amount > 0
 139 |  IERC20(swapParams.tokenIn).transferFrom(
 140 |      msg.sender,
 141 |      address(this),
 142 |      swapParams.amountIn
 143 |  );
```
GitHub: [109-113](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L109-L113), [139-143](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L139-L143)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Check transfer amount > 0
  88 |  IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);
```
GitHub: [88](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Check transfer amount > 0
  44 |  IERC20(token).transfer(user, amount);

     |  // @audit-issue Check transfer amount > 0
  55 |  IERC20(token).transfer(recipient, amount);

     |  // @audit-issue Check transfer amount > 0
  83 |  IERC20(swapParams.tokenIn).transferFrom(
  84 |      msg.sender,
  85 |      address(this),
  86 |      swapParams.amountIn
  87 |  );
```
GitHub: [44](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L44), [55](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L55), [83-87](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L83-L87)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Check transfer amount > 0
 181 |  weth.transferFrom(msg.sender, address(this), _amount);

     |  // @audit-issue Check transfer amount > 0
 267 |  dcntEth.transfer(_to, _amount);

     |  // @audit-issue Check transfer amount > 0
 273 |  weth.transfer(_to, _amount);

     |  // @audit-issue Check transfer amount > 0
 288 |  dcntEth.transferFrom(msg.sender, address(this), amount);

     |  // @audit-issue Check transfer amount > 0
 297 |  dcntEth.transferFrom(msg.sender, address(this), amount);

     |  // @audit-issue Check transfer amount > 0
 298 |  weth.transfer(msg.sender, amount);

     |  // @audit-issue Check transfer amount > 0
 325 |  weth.transferFrom(msg.sender, address(this), amount);

     |  // @audit-issue Check transfer amount > 0
 334 |  weth.transfer(msg.sender, amount);
```
GitHub: [181](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L181), [267](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L267), [273](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L273), [288](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L288), [297](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L297), [298](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L298), [325](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L325), [334](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L334)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue Check transfer amount > 0
  36 |  weth.transfer(from, amount);

     |  // @audit-issue Check transfer amount > 0
  44 |  weth.transfer(from, remainingAfterCall);

     |  // @audit-issue Check transfer amount > 0
  75 |  weth.transferFrom(msg.sender, address(this), amount);
```
GitHub: [36](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L36), [44](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L44), [75](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L75)


## [G-03] Cache multiple accesses of mapping/array values

Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata.

*Instances: 2*

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-info Contains multiple accesses to state
  81 |  function bridge(
  82 |      uint256 amt2Bridge,
  83 |      SwapInstructions memory postBridge,
  84 |      uint256 dstChainId,
  85 |      address target,
  86 |      address paymentOperator,
  87 |      bytes memory payload,
  88 |      bytes calldata additionalArgs,
  89 |      address payable refund
  90 |  ) public payable onlyUtb returns (bytes memory bridgePayload) {
  :  |
     |          // @audit-issue Cache this value
  92 |          destinationBridgeAdapter[dstChainId] != address(0),
  :  |
     |          // @audit-issue Cache this value
 119 |          destinationBridgeAdapter[dstChainId],
```
GitHub: [92](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L92), [119](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L119)


## [G-04] Combine multiple mappings with the same key type where appropriate to reduce gas usage

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

*Instances: 2*

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Consider whether multiple mappings can be merged into a single struct mapping
  25 |  mapping(uint256 => address) public destinationBridgeAdapter;

     |  // @audit-issue Consider whether multiple mappings can be merged into a single struct mapping
  26 |  mapping(uint256 => uint16) lzIdLookup;
```
GitHub: [25](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L25), [26](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L26)


## [G-05] Consider changing some `public` variables to `private`/`internal`

Public state variables in Solidity automatically generate getter functions, increasing deployment costs.
                    
Examples of variables that probably don't need to be public - anybody who needs to inspect them can check the on-chain contract storage:

* Factories
* Controllers
* Governance
* Owner, roles, etc.



*Instances: 25*

```solidity
File: src/UTB.sol

     |  // @audit-issue Reconsider whether this variable has to be public
  21 |  mapping(uint8 => address) public swappers;

     |  // @audit-issue Reconsider whether this variable has to be public
  22 |  mapping(uint8 => address) public bridgeAdapters;
```
GitHub: [21](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L21), [22](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L22)


```solidity
File: src/bridge_adapters/BaseAdapter.sol

     |  // @audit-issue Reconsider whether this variable has to be public
   7 |  address public bridgeExecutor;
```
GitHub: [7](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L7)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue Reconsider whether this variable has to be public
  13 |  uint8 public constant BRIDGE_ID = 0;

     |  // @audit-issue Reconsider whether this variable has to be public
  14 |  mapping(uint256 => address) public destinationBridgeAdapter;

     |  // @audit-issue Reconsider whether this variable has to be public
  15 |  IDecentEthRouter public router;
```
GitHub: [13](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L13), [14](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L14), [15](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L15)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Reconsider whether this variable has to be public
  22 |  uint8 public constant BRIDGE_ID = 1;

     |  // @audit-issue Reconsider whether this variable has to be public
  23 |  uint8 public constant SG_FEE_BPS = 6;

     |  // @audit-issue Reconsider whether this variable has to be public
  24 |  address public stargateEth;

     |  // @audit-issue Reconsider whether this variable has to be public
  25 |  mapping(uint256 => address) public destinationBridgeAdapter;

     |  // @audit-issue Reconsider whether this variable has to be public
  31 |  IStargateRouter public router;
```
GitHub: [22](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L22), [23](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L23), [24](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L24), [25](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L25), [31](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L31)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Reconsider whether this variable has to be public
  16 |  uint8 public constant SWAPPER_ID = 0;

     |  // @audit-issue Reconsider whether this variable has to be public
  17 |  address public uniswap_router;

     |  // @audit-issue Reconsider whether this variable has to be public
  18 |  address payable public wrapped;
```
GitHub: [16](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L16), [17](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L17), [18](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L18)


```solidity
File: lib/decent-bridge/src/DcntEth.sol

     |  // @audit-issue Reconsider whether this variable has to be public
   6 |  address public router;
```
GitHub: [6](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L6)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Reconsider whether this variable has to be public
  13 |  IWETH public weth;

     |  // @audit-issue Reconsider whether this variable has to be public
  14 |  IDcntEth public dcntEth;

     |  // @audit-issue Reconsider whether this variable has to be public
  15 |  IDecentBridgeExecutor public executor;

     |  // @audit-issue Reconsider whether this variable has to be public
  17 |  uint8 public constant MT_ETH_TRANSFER = 0;

     |  // @audit-issue Reconsider whether this variable has to be public
  18 |  uint8 public constant MT_ETH_TRANSFER_WITH_PAYLOAD = 1;

     |  // @audit-issue Reconsider whether this variable has to be public
  20 |  uint16 public constant PT_SEND_AND_CALL = 1;

     |  // @audit-issue Reconsider whether this variable has to be public
  22 |  bool public gasCurrencyIsEth; // for chains that use ETH as gas currency

     |  // @audit-issue Reconsider whether this variable has to be public
  24 |  mapping(uint16 => address) public destinationBridges;

     |  // @audit-issue Reconsider whether this variable has to be public
  25 |  mapping(address => uint256) public balanceOf;
```
GitHub: [13](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L13), [14](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L14), [15](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L15), [17](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L17), [18](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L18), [20](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L20), [22](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L22), [24](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L24), [25](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L25)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue Reconsider whether this variable has to be public
  10 |  bool public gasCurrencyIsEth; // for chains that use ETH as gas currency
```
GitHub: [10](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L10)


## [G-06] Function result should be cached

The instances below show multiple calls to a single function within the same function.

*Instances: 4*

```solidity
File: src/UTBExecutor.sol

     |  // @audit-info Contains multiple external contract reads
  41 |  function execute(
  42 |      address target,
  43 |      address paymentOperator,
  44 |      bytes memory payload,
  45 |      address token,
  46 |      uint amount,
  47 |      address payable refund,
  48 |      uint extraNative
  49 |  ) public onlyOwner {
  :  |
     |      // @audit-issue Cache this value
  59 |      uint initBalance = IERC20(token).balanceOf(address(this));
  :  |
     |      // @audit-issue Cache this value
  73 |      uint remainingBalance = IERC20(token).balanceOf(address(this)) -
```
GitHub: [59](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L59), [73](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L73)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-info Contains multiple external contract reads
  24 |  function _executeWeth(
  25 |      address from,
  26 |      address target,
  27 |      uint256 amount,
  28 |      bytes memory callPayload
  29 |  ) private {
     |      // @audit-issue Cache this value
  30 |      uint256 balanceBefore = weth.balanceOf(address(this));
  :  |
     |          // @audit-issue Cache this value
  41 |          (balanceBefore - weth.balanceOf(address(this)));
```
GitHub: [30](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L30), [41](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L41)


## [G-07] Inline `modifier`s that are only used once, to save gas

Inline `modifier`s that are only used once, to save gas.

*Instances: 9*

```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-info Contains redundant modifiers
   8 |  contract UTBFeeCollector is UTBOwned {
  :  |
     |      // @audit-issue This is the only invocation of this modifier
  48 |      ) public payable onlyUtb {
```
GitHub: [48](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L48)


```solidity
File: src/bridge_adapters/BaseAdapter.sol

     |  // @audit-info Contains redundant modifiers
   6 |  contract BaseAdapter is UTBOwned {
  :  |
     |      // @audit-issue This is the only invocation of this modifier
  19 |      function setBridgeExecutor(address _executor) public onlyOwner {
```
GitHub: [19](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L19)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-info Contains redundant modifiers
  12 |  contract DecentBridgeAdapter is BaseAdapter, IBridgeAdapter {
  :  |
     |      // @audit-issue This is the only invocation of this modifier
  90 |      ) public payable onlyUtb returns (bytes memory bridgePayload) {
  :  |
     |      // @audit-issue This is the only invocation of this modifier
 133 |      ) public onlyExecutor {
```
GitHub: [90](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L90), [133](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L133)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-info Contains redundant modifiers
  17 |  contract StargateBridgeAdapter is
  :  |
     |      // @audit-issue This is the only invocation of this modifier
  78 |      ) public payable onlyUtb returns (bytes memory bridgePayload) {
  :  |
     |      // @audit-issue This is the only invocation of this modifier
 190 |      ) external override onlyExecutor {
```
GitHub: [78](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L78), [190](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L190)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-info Contains redundant modifiers
  13 |  contract UniSwapper is UTBOwned, ISwapper {
  :  |
     |          // @audit-issue This is the only invocation of this modifier
  62 |          onlyUtb
```
GitHub: [62](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L62)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-info Contains redundant modifiers
  12 |  contract DecentEthRouter is IDecentEthRouter, IOFTReceiverV2, Owned {
  :  |
     |      // @audit-issue This is the only invocation of this modifier
 244 |      ) external override onlyLzApp {
```
GitHub: [244](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L244)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-info Contains redundant modifiers
   8 |  contract DecentBridgeExecutor is IDecentBridgeExecutor, Owned {
  :  |
     |      // @audit-issue This is the only invocation of this modifier
  74 |      ) public onlyOwner {
```
GitHub: [74](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L74)


## [G-08] Mutable state variables explicitly initialized to zero consume gas unnecessarily

The default value for variables is zero, so initializing them to zero is superfluous. This will also save gas if the variable is a mutable state variable.

*Instances: 1*

```solidity
File: src/UTB.sol

     |  // @audit-issue Initial value is unnecessary
 234 |  uint value = 0;
```
GitHub: [234](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L234)


## [G-09] Not using the named return variable is confusing and can waste gas

Consider changing the variable to be an unnamed one, since the variable is never assigned, nor is it returned by name. If the optimizer is not turned on, leaving the code as it is will also waste gas for the stack variable.

*Instances: 10*

```solidity
File: src/UTB.sol

     |  // @audit-issue Remove unused variable `tokenOut` to save gas
     |  // @audit-issue Remove unused variable `amountOut` to save gas
  54 |  ) private returns (address tokenOut, uint256 amountOut) {
```
GitHub: [54](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L54), [54](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L54)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue Remove unused variable `nativeFee` to save gas
     |  // @audit-issue Remove unused variable `zroFee` to save gas
  50 |  ) public view returns (uint nativeFee, uint zroFee) {
```
GitHub: [50](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L50), [50](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L50)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Remove unused variable `value` to save gas
 103 |  ) private view returns (uint value) {
```
GitHub: [103](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L103)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Remove unused variable `_swapParams` to save gas
  81 |  ) private returns (SwapParams memory _swapParams) {

     |  // @audit-issue Remove unused variable `tokenOut` to save gas
     |  // @audit-issue Remove unused variable `amountOut` to save gas
 104 |  ) public payable returns (address tokenOut, uint256 amountOut) {
```
GitHub: [81](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L81), [104](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L104), [104](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L104)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Remove unused variable `nativeFee` to save gas
     |  // @audit-issue Remove unused variable `zroFee` to save gas
 121 |  ) public view returns (uint nativeFee, uint zroFee) {
```
GitHub: [121](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L121), [121](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L121)


## [G-10] Public functions not used internally can be marked as external to save gas

Public functions that aren't used internally in Solidity contracts should be made external to optimize gas usage and improve contract efficiency. External functions can only be called from outside the contract, and their arguments are directly read from the calldata, which is more gas-efficient than loading them into memory, as is the case for public functions. By using external visibility, developers can reduce gas consumption for external calls and ensure that the contract operates more cost-effectively for users. Moreover, setting the appropriate visibility level for functions also enhances code readability and maintainability, promoting a more secure and well-structured contract design. 

*Instances: 44*

```solidity
File: src/UTB.sol

     |  // @audit-info Context
  15 |  contract UTB is Owned {
  :  |
     |      // @audit-issue Not called internally by this contract
  28 |      function setExecutor(address _executor) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
  36 |      function setWrapped(address payable _wrapped) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
  44 |      function setFeeCollector(address payable _feeCollector) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
 108 |      function swapAndExecute(
 109 |          SwapAndExecuteInstructions calldata instructions,
 110 |          FeeStructure calldata fees,
 111 |          bytes calldata signature
 112 |      )
 113 |          public
 114 |          payable
 115 |          retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
  :  |
     |      // @audit-issue Not called internally by this contract
 259 |      function bridgeAndExecute(
 260 |          BridgeInstructions calldata instructions,
 261 |          FeeStructure calldata fees,
 262 |          bytes calldata signature
 263 |      )
 264 |          public
 265 |          payable
 266 |          retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
 267 |          returns (bytes memory)
  :  |
     |      // @audit-issue Not called internally by this contract
 311 |      function receiveFromBridge(
 312 |          SwapInstructions memory postBridge,
 313 |          address target,
 314 |          address paymentOperator,
 315 |          bytes memory payload,
 316 |          address payable refund
 317 |      ) public {
  :  |
     |      // @audit-issue Not called internally by this contract
 325 |      function registerSwapper(address swapper) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
 334 |      function registerBridge(address bridge) public onlyOwner {
```
GitHub: [28](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L28), [36](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L36), [44](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L44), [108-116](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L108-L116), [259-268](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L259-L268), [311-317](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L311-L317), [325](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L325), [334](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L334)


```solidity
File: src/UTBExecutor.sol

     |  // @audit-info Context
   7 |  contract UTBExecutor is Owned {
  :  |
     |      // @audit-issue Not called internally by this contract
  19 |      function execute(
  20 |          address target,
  21 |          address paymentOperator,
  22 |          bytes memory payload,
  23 |          address token,
  24 |          uint amount,
  25 |          address payable refund
  26 |      ) public payable onlyOwner {
```
GitHub: [19-26](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L19-L26)


```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-info Context
   8 |  contract UTBFeeCollector is UTBOwned {
  :  |
     |      // @audit-issue Not called internally by this contract
  18 |      function setSigner(address _signer) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
  44 |      function collectFees(
  45 |          FeeStructure calldata fees,
  46 |          bytes memory packedInfo,
  47 |          bytes memory signature
  48 |      ) public payable onlyUtb {
  :  |
     |      // @audit-issue Not called internally by this contract
  69 |      function redeemFees(address token, uint amount) public onlyOwner {
```
GitHub: [18](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L18), [44-48](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L44-L48), [69](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L69)


```solidity
File: src/bridge_adapters/BaseAdapter.sol

     |  // @audit-info Context
   6 |  contract BaseAdapter is UTBOwned {
  :  |
     |      // @audit-issue Not called internally by this contract
  19 |      function setBridgeExecutor(address _executor) public onlyOwner {
```
GitHub: [19](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L19)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-info Context
  12 |  contract DecentBridgeAdapter is BaseAdapter, IBridgeAdapter {
  :  |
     |      // @audit-issue Not called internally by this contract
  26 |      function setRouter(address _router) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
  30 |      function getId() public pure returns (uint8) {
  :  |
     |      // @audit-issue Not called internally by this contract
  34 |      function registerRemoteBridgeAdapter(
  35 |          uint256 dstChainId,
  36 |          uint16 dstLzId,
  37 |          address decentBridgeAdapter
  38 |      ) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
  44 |      function estimateFees(
  45 |          SwapInstructions memory postBridge,
  46 |          uint256 dstChainId,
  47 |          address target,
  48 |          uint64 dstGas,
  49 |          bytes memory payload
  50 |      ) public view returns (uint nativeFee, uint zroFee) {
  :  |
     |      // @audit-issue Not called internally by this contract
  81 |      function bridge(
  82 |          uint256 amt2Bridge,
  83 |          SwapInstructions memory postBridge,
  84 |          uint256 dstChainId,
  85 |          address target,
  86 |          address paymentOperator,
  87 |          bytes memory payload,
  88 |          bytes calldata additionalArgs,
  89 |          address payable refund
  90 |      ) public payable onlyUtb returns (bytes memory bridgePayload) {
  :  |
     |      // @audit-issue Not called internally by this contract
 127 |      function receiveFromBridge(
 128 |          SwapInstructions memory postBridge,
 129 |          address target,
 130 |          address paymentOperator,
 131 |          bytes memory payload,
 132 |          address payable refund
 133 |      ) public onlyExecutor {
```
GitHub: [26](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L26), [30](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L30), [34-38](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L34-L38), [44-50](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L44-L50), [81-90](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L81-L90), [127-133](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L127-L133)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-info Context
  17 |  contract StargateBridgeAdapter is
  :  |
     |      // @audit-issue Not called internally by this contract
  33 |      function setRouter(address _router) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
  37 |      function setStargateEth(address _sgEth) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
  41 |      function getId() public pure returns (uint8) {
  :  |
     |      // @audit-issue Not called internally by this contract
  45 |      function registerRemoteBridgeAdapter(
  46 |          uint256 dstChainId,
  47 |          uint16 dstLzId,
  48 |          address decentBridgeAdapter
  49 |      ) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
  69 |      function bridge(
  70 |          uint256 amt2Bridge,
  71 |          SwapInstructions memory postBridge,
  72 |          uint256 dstChainId,
  73 |          address target,
  74 |          address paymentOperator,
  75 |          bytes memory payload,
  76 |          bytes calldata additionalArgs,
  77 |          address payable refund
  78 |      ) public payable onlyUtb returns (bytes memory bridgePayload) {
```
GitHub: [33](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L33), [37](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L37), [41](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L41), [45-49](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L45-L49), [69-78](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L69-L78)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-info Context
  13 |  contract UniSwapper is UTBOwned, ISwapper {
  :  |
     |      // @audit-issue Not called internally by this contract
  20 |      function setRouter(address _router) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
  24 |      function setWrapped(address payable _wrapped) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
  28 |      function getId() public pure returns (uint8) {
```
GitHub: [20](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L20), [24](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L24), [28](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L28)


```solidity
File: lib/decent-bridge/src/DcntEth.sol

     |  // @audit-info Context
   5 |  contract DcntEth is OFTV2 {
  :  |
     |      // @audit-issue Not called internally by this contract
  20 |      function setRouter(address _router) public {
  :  |
     |      // @audit-issue Not called internally by this contract
  24 |      function mint(address _to, uint256 _amount) public onlyRouter {
  :  |
     |      // @audit-issue Not called internally by this contract
  28 |      function burn(address _from, uint256 _amount) public onlyRouter {
  :  |
     |      // @audit-issue Not called internally by this contract
  32 |      function mintByOwner(address _to, uint256 _amount) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
  36 |      function burnByOwner(address _from, uint256 _amount) public onlyOwner {
```
GitHub: [20](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L20), [24](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L24), [28](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L28), [32](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L32), [36](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L36)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-info Context
  12 |  contract DecentEthRouter is IDecentEthRouter, IOFTReceiverV2, Owned {
  :  |
     |      // @audit-issue Not called internally by this contract
  68 |      function registerDcntEth(address _addr) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
  73 |      function addDestinationBridge(
  74 |          uint16 _dstChainId,
  75 |          address _routerAddress
  76 |      ) public onlyOwner {
  :  |
     |      // @audit-issue Not called internally by this contract
 113 |      function estimateSendAndCallFee(
 114 |          uint8 msgType,
 115 |          uint16 _dstChainId,
 116 |          address _toAddress,
 117 |          uint _amount,
 118 |          uint64 _dstGasForCall,
 119 |          bool deliverEth,
 120 |          bytes memory payload
 121 |      ) public view returns (uint nativeFee, uint zroFee) {
  :  |
     |      // @audit-issue Not called internally by this contract
 197 |      function bridgeWithPayload(
 198 |          uint16 _dstChainId,
 199 |          address _toAddress,
 200 |          uint _amount,
 201 |          bool deliverEth,
 202 |          uint64 _dstGasForCall,
 203 |          bytes memory additionalPayload
 204 |      ) public payable {
  :  |
     |      // @audit-issue Not called internally by this contract
 218 |      function bridge(
 219 |          uint16 _dstChainId,
 220 |          address _toAddress,
 221 |          uint _amount,
 222 |          uint64 _dstGasForCall,
 223 |          bool deliverEth // if false, delivers WETH
 224 |      ) public payable {
  :  |
     |      // @audit-issue Not called internally by this contract
 285 |      function redeemEth(
 286 |          uint256 amount
 287 |      ) public onlyIfWeHaveEnoughReserves(amount) {
  :  |
     |      // @audit-issue Not called internally by this contract
 294 |      function redeemWeth(
 295 |          uint256 amount
 296 |      ) public onlyIfWeHaveEnoughReserves(amount) {
  :  |
     |      // @audit-issue Not called internally by this contract
 302 |      function addLiquidityEth()
 303 |          public
 304 |          payable
 305 |          onlyEthChain
 306 |          userDepositing(msg.value)
  :  |
     |      // @audit-issue Not called internally by this contract
 313 |      function removeLiquidityEth(
 314 |          uint256 amount
 315 |      ) public onlyEthChain userIsWithdrawing(amount) {
  :  |
     |      // @audit-issue Not called internally by this contract
 322 |      function addLiquidityWeth(
 323 |          uint256 amount
 324 |      ) public payable userDepositing(amount) {
  :  |
     |      // @audit-issue Not called internally by this contract
 330 |      function removeLiquidityWeth(
 331 |          uint256 amount
 332 |      ) public userIsWithdrawing(amount) {
```
GitHub: [68](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L68), [73-76](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L73-L76), [113-121](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L113-L121), [197-204](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L197-L204), [218-224](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L218-L224), [285-287](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L285-L287), [294-296](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L294-L296), [302-307](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L302-L307), [313-315](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L313-L315), [322-324](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L322-L324), [330-332](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L330-L332)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-info Context
   8 |  contract DecentBridgeExecutor is IDecentBridgeExecutor, Owned {
  :  |
     |      // @audit-issue Not called internally by this contract
  68 |      function execute(
  69 |          address from,
  70 |          address target,
  71 |          bool deliverEth,
  72 |          uint256 amount,
  73 |          bytes memory callPayload
  74 |      ) public onlyOwner {
```
GitHub: [68-74](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L68-L74)


## [G-11] Reduce deployment gas costs by fine-tuning IPFS file hashes

Solc [currently by default](https://docs.soliditylang.org/en/v0.8.23/metadata.html#encoding-of-the-metadata-hash-in-the-bytecode) appends the IPFS hash (in CID v0) of the canonical metadata file and the compiler version to the end of the bytecode. This value is variable-length [CBOR-encoded](https://tools.ietf.org/html/rfc7049) i.e. it can be optimized in order to reduce deployment gas costs.
                    
*Note:* multiple contracts in the same file will share the same hash.

*Instances: 11*

```solidity
File: src/UTB.sol

     |  // @audit-issue IPFS hash is dweb:/ipfs/Qmb4wNnaw7TpYMupEzTjXC2sVCisCZ75ecGWJRarfBjyjr
  15 |  contract UTB is Owned {
```
GitHub: [15](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L15)


```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue IPFS hash is dweb:/ipfs/QmNZfFvkgDdvQJDJUEWi3nJfUBGFrzccQPxrekwP2z7oFK
   7 |  contract UTBExecutor is Owned {
```
GitHub: [7](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L7)


```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-issue IPFS hash is dweb:/ipfs/Qma7X68CrMzTbHBAKNoYh9CVk46cG1dnpSdG68J3isgJPv
   8 |  contract UTBFeeCollector is UTBOwned {
```
GitHub: [8](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L8)


```solidity
File: src/bridge_adapters/BaseAdapter.sol

     |  // @audit-issue IPFS hash is dweb:/ipfs/QmQWkzewFaXUhZKTmMa3qStF1JK24zyTGvwHp4j7cCWipj
   6 |  contract BaseAdapter is UTBOwned {
```
GitHub: [6](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L6)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue IPFS hash is dweb:/ipfs/QmTWYgGshX2CgMUd7ChLXSH79ADzt3ifjdP8hbJxyCiosi
  12 |  contract DecentBridgeAdapter is BaseAdapter, IBridgeAdapter {
```
GitHub: [12](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L12)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue IPFS hash is dweb:/ipfs/QmcQxuCDpLPs3u61o3HanuMjXyKnd6gbvJi6iX2QoL63vV
  17 |  contract StargateBridgeAdapter is
```
GitHub: [17](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L17)


```solidity
File: src/swappers/SwapParams.sol

     |  // @audit-issue IPFS hash is dweb:/ipfs/QmacYtBp4g9bGVKVPuFcVQzXRqhHn7rQHzEePx4ZhCKqW8
   4 |  library SwapDirection {
```
GitHub: [4](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L4)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue IPFS hash is dweb:/ipfs/QmT5DABeQ9LuoTGahu5qePA2MjGke5TmMZB2bG7NeeLGLb
  13 |  contract UniSwapper is UTBOwned, ISwapper {
```
GitHub: [13](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L13)


```solidity
File: lib/decent-bridge/src/DcntEth.sol

     |  // @audit-issue IPFS hash is dweb:/ipfs/QmVZso3k2DpezXM27TcGKzw2rrVDSm6qAY8xi32MTtsdD7
   5 |  contract DcntEth is OFTV2 {
```
GitHub: [5](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DcntEth.sol#L5)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue IPFS hash is dweb:/ipfs/QmbMTDoVp8i4k6hSH317YTqMWsEyzuZHZ9QkV54f35YkXX
  12 |  contract DecentEthRouter is IDecentEthRouter, IOFTReceiverV2, Owned {
```
GitHub: [12](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L12)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue IPFS hash is dweb:/ipfs/QmXDJNNP7kdYAhVbwqzdXKFuzZfPuL1v9AUkqFQhB3Sc9u
   8 |  contract DecentBridgeExecutor is IDecentBridgeExecutor, Owned {
```
GitHub: [8](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L8)


## [G-12] Redundant type conversion

Casting a variable to its own type is redundant and a waste of gas.

*Instances: 1*

```solidity
File: src/bridge_adapters/BaseAdapter.sol

     |  // @audit-issue `address(bridgeExecutor)` is a redundant type cast
  13 |  msg.sender == address(bridgeExecutor),
```
GitHub: [13](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L13)


## [G-13] Same cast is done multiple times

It's cheaper to do it once, and store the result to a variable.

*Instances: 13*

```solidity
File: src/UTB.sol

     |  // @audit-info Contains redundant type conversions
  63 |  function performSwap(
  64 |      SwapInstructions memory swapInstructions,
  65 |      bool retrieveTokenIn
  66 |  ) private returns (address tokenOut, uint256 amountOut) {
  :  |
     |          // @audit-issue Cache this type conversion result that is used in multiple locations
  83 |          IERC20(swapParams.tokenIn).transferFrom(
  :  |
     |      // @audit-issue Cache this type conversion result that is used in multiple locations
  90 |      IERC20(swapParams.tokenIn).approve(
```
GitHub: [83](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L83), [90](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L90)


```solidity
File: src/UTBExecutor.sol

     |  // @audit-info Contains redundant type conversions
  41 |  function execute(
  42 |      address target,
  43 |      address paymentOperator,
  44 |      bytes memory payload,
  45 |      address token,
  46 |      uint amount,
  47 |      address payable refund,
  48 |      uint extraNative
  49 |  ) public onlyOwner {
  :  |
     |      // @audit-issue Cache this type conversion result that is used in multiple locations
  59 |      uint initBalance = IERC20(token).balanceOf(address(this));
  :  |
     |      // @audit-issue Cache this type conversion result that is used in multiple locations
  61 |      IERC20(token).transferFrom(msg.sender, address(this), amount);
     |      // @audit-issue Cache this type conversion result that is used in multiple locations
  62 |      IERC20(token).approve(paymentOperator, amount);
  :  |
     |      // @audit-issue Cache this type conversion result that is used in multiple locations
  73 |      uint remainingBalance = IERC20(token).balanceOf(address(this)) -
  :  |
     |      // @audit-issue Cache this type conversion result that is used in multiple locations
  80 |      IERC20(token).transfer(refund, remainingBalance);
```
GitHub: [59](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L59), [61](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L61), [62](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L62), [73](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L73), [80](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L80)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-info Contains redundant type conversions
  81 |  function bridge(
  82 |      uint256 amt2Bridge,
  83 |      SwapInstructions memory postBridge,
  84 |      uint256 dstChainId,
  85 |      address target,
  86 |      address paymentOperator,
  87 |      bytes memory payload,
  88 |      bytes calldata additionalArgs,
  89 |      address payable refund
  90 |  ) public payable onlyUtb returns (bytes memory bridgePayload) {
  :  |
     |          // @audit-issue Cache this type conversion result that is used in multiple locations
 109 |          IERC20(bridgeToken).transferFrom(
  :  |
     |          // @audit-issue Cache this type conversion result that is used in multiple locations
 114 |          IERC20(bridgeToken).approve(address(router), amt2Bridge);

     |  // @audit-info Contains redundant type conversions
 127 |  function receiveFromBridge(
 128 |      SwapInstructions memory postBridge,
 129 |      address target,
 130 |      address paymentOperator,
 131 |      bytes memory payload,
 132 |      address payable refund
 133 |  ) public onlyExecutor {
  :  |
     |      // @audit-issue Cache this type conversion result that is used in multiple locations
 139 |      IERC20(swapParams.tokenIn).transferFrom(
  :  |
     |      // @audit-issue Cache this type conversion result that is used in multiple locations
 145 |      IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn);
```
GitHub: [109](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L109), [114](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L114), [139](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L139), [145](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L145)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-info Contains redundant type conversions
  69 |  function bridge(
  70 |      uint256 amt2Bridge,
  71 |      SwapInstructions memory postBridge,
  72 |      uint256 dstChainId,
  73 |      address target,
  74 |      address paymentOperator,
  75 |      bytes memory payload,
  76 |      bytes calldata additionalArgs,
  77 |      address payable refund
  78 |  ) public payable onlyUtb returns (bytes memory bridgePayload) {
  :  |
     |      // @audit-issue Cache this type conversion result that is used in multiple locations
  88 |      IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);
     |      // @audit-issue Cache this type conversion result that is used in multiple locations
  89 |      IERC20(bridgeToken).approve(address(router), amt2Bridge);
```
GitHub: [88](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88), [89](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L89)


## [G-14] Simple checks for zero can be done using assembly to save gas

Using assembly for simple zero checks on unsigned integers can save gas due to lower-level, optimized operations. 

*Instances: 16*

```solidity
File: src/UTB.sol

     |  // @audit-issue Use inline assembly
  74 |  if (swapParams.tokenIn == address(0)) {

     |  // @audit-issue Use inline assembly
  97 |  if (tokenOut == address(0)) {

     |  // @audit-issue Use inline assembly
 142 |  if (tokenOut == address(0)) {

     |  // @audit-issue Use inline assembly
 215 |  if (bridgeToken != address(0)) {

     |  // @audit-issue Use inline assembly
 233 |  if (address(feeCollector) != address(0)) {

     |  // @audit-issue Use inline assembly
 235 |  if (fees.feeToken != address(0)) {
```
GitHub: [74](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L74), [97](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L97), [142](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L142), [215](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L215), [233](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L233), [235](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L235)


```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Use inline assembly
  51 |  if (token == address(0)) {

     |  // @audit-issue Use inline assembly
  76 |  if (remainingBalance == 0) {
```
GitHub: [51](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L51), [76](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L76)


```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-issue Use inline assembly
  55 |  if (fees.feeToken != address(0)) {

     |  // @audit-issue Use inline assembly
  70 |  if (token == address(0)) {
```
GitHub: [55](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L55), [70](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L70)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue Use inline assembly
  92 |  destinationBridgeAdapter[dstChainId] != address(0),
```
GitHub: [92](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L92)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Use inline assembly
 158 |  dstAddr != address(0),
```
GitHub: [158](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L158)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Use inline assembly
  52 |  if (token == address(0)) {

     |  // @audit-issue Use inline assembly
  68 |  if (swapParams.path.length == 0) {

     |  // @audit-issue Use inline assembly
  82 |  if (swapParams.tokenIn != address(0)) {

     |  // @audit-issue Use inline assembly
  96 |  require(uniswap_router != address(0), "router not set");
```
GitHub: [52](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L52), [68](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L68), [82](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L82), [96](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L96)


## [G-15] Subtraction can potentially be marked `unchecked` to save gas

In Solidity 0.8.x and above, arithmetic operations like subtraction automatically check for underflows and overflows, and revert the transaction if such a condition is met. This built-in safety feature provides a layer of security against potential numerical errors. However, these automatic checks also come with additional gas costs.

In some situations, you may already have a guard condition, like a require() statement or an if statement, that ensures the safety of the arithmetic operation. In such cases, the automatic check becomes redundant and leads to unnecessary gas expenditure.

For example, you may have a function that subtracts a smaller number from a larger one, and you may have already verified that the smaller number is indeed smaller. In this case, you're already sure that the subtraction operation won't underflow, so there's no need for the automatic check.

In these situations, you can use the unchecked { } block around the subtraction operation to skip the automatic check. This will reduce gas costs and make your contract more efficient, without sacrificing security. However, it's critical to use unchecked { } only when you're absolutely sure that the operation is safe.

*Instances: 8*

```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Use unchecked to save gas, if possible
  73 |  uint remainingBalance = IERC20(token).balanceOf(address(this)) -
  74 |      initBalance;
```
GitHub: [73-74](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L73-L74)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Use unchecked to save gas, if possible
  66 |  return (amt2Bridge * (1e4 - SG_FEE_BPS)) / 1e4;

     |  // @audit-issue Use unchecked to save gas, if possible
 176 |  (amt2Bridge * (10000 - SG_FEE_BPS)) / 10000, // the min qty you would accept on the destination, fee is 6 bips
```
GitHub: [66](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L66), [176](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L176)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Use unchecked to save gas, if possible
 111 |  swapParams.amountIn - swapParams.amountOut

     |  // @audit-issue Use unchecked to save gas, if possible
 165 |  params.amountInMaximum - amountIn
```
GitHub: [111](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L111), [165](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L165)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Use unchecked to save gas, if possible
 179 |  gasValue = msg.value - _amount;
```
GitHub: [179](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L179)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue Use unchecked to save gas, if possible
  40 |  uint256 remainingAfterCall = amount -
  41 |      (balanceBefore - weth.balanceOf(address(this)));

     |  // @audit-issue Use unchecked to save gas, if possible
  41 |  (balanceBefore - weth.balanceOf(address(this)));
```
GitHub: [40-41](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L40-L41), [41](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L41)


## [G-16] The result of function calls should be cached rather than re-calling the function

The instances below point to the second+ call of the function within a single function.

*Instances: 4*

```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Called 2 times in execute()
  59 |  uint initBalance = IERC20(token).balanceOf(address(this));

     |  // @audit-issue Called 2 times in execute()
  73 |  uint remainingBalance = IERC20(token).balanceOf(address(this)) -
```
GitHub: [59](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L59), [73](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L73)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue Called 2 times in _executeWeth()
  30 |  uint256 balanceBefore = weth.balanceOf(address(this));

     |  // @audit-issue Called 2 times in _executeWeth()
  41 |  (balanceBefore - weth.balanceOf(address(this)));
```
GitHub: [30](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L30), [41](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L41)


## [G-17] Use != 0 instead of > 0

If possible, i.e. the range of the variable precludes negative values, consider using `!= 0` to save gas.

*Instances: 1*

```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Use != 0 instead
  64 |  if (extraNative > 0) {
```
GitHub: [64](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L64)


## [G-18] Use Solady library where possible to save gas

Utilizing gas-optimized math functions from libraries like [Solady](https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol) can lead to more efficient smart contracts.
This is particularly beneficial in contracts where these operations are frequently used.

For example, `(x * WAD) / y` can be replaced with Solady's `divWad()`.


*Instances: 2*

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Consider using the Solady library for this arithmetic
  66 |  return (amt2Bridge * (1e4 - SG_FEE_BPS)) / 1e4;

     |  // @audit-issue Consider using the Solady library for this arithmetic
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


## [G-19] Use assembly to calculate hashes

If the arguments to the encode call can fit into the scratch space (two words or fewer), then it's more efficient to use assembly to generate the hash (**80 gas**):
                    
```solidity
    keccak256(abi.encodePacked(a, b)); }
```

to

```solidity 
    assembly {
        mstore(0x00, a)
        mstore(0x20, b)
        let result := keccak256(0x00, 0x40)
    }
```

*Instances: 2*

```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-issue Use assembly to calcuate hashes
  49 |  bytes32 constructedHash = keccak256(
  50 |      abi.encodePacked(BANNER, keccak256(packedInfo))
  51 |  );

     |  // @audit-issue Use assembly to calcuate hashes
  50 |  abi.encodePacked(BANNER, keccak256(packedInfo))
```
GitHub: [49-51](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L49-L51), [50](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L50)


## [G-20] Use assembly to perform external calls, in order to save gas

Using Solidity's assembly scratch space for constructing calldata in external calls with one or two arguments can be a gas-efficient approach. This method leverages the designated memory area (the first 64 bytes of memory) for temporary data storage during assembly operations. By directly writing arguments into this scratch space, it eliminates the need for additional memory allocation typically required for calldata preparation. This technique can lead to notable gas savings, especially in high-frequency or gas-sensitive operations. However, it requires careful implementation to avoid data corruption and should be used with a thorough understanding of low-level EVM operations and memory handling. Proper testing and validation are crucial when employing such optimizations.

*Instances: 51*

```solidity
File: src/UTB.sol

     |  // @audit-issue Save gas by call this in assembly
  76 |  wrapped.deposit{value: swapParams.amountIn}();

     |  // @audit-issue Save gas by call this in assembly
  78 |  swapInstructions.swapPayload = swapper.updateSwapParams(
  79 |      swapParams,
  80 |      swapInstructions.swapPayload
  81 |  );

     |  // @audit-issue Save gas by call this in assembly
  90 |  IERC20(swapParams.tokenIn).approve(
  91 |      address(swapper),
  92 |      swapParams.amountIn
  93 |  );

     |  // @audit-issue Save gas by call this in assembly
  95 |  (tokenOut, amountOut) = swapper.swap(swapInstructions.swapPayload);

     |  // @audit-issue Save gas by call this in assembly
  98 |  wrapped.withdraw(amountOut);

     |  // @audit-issue Save gas by call this in assembly
 152 |  IERC20(tokenOut).approve(address(executor), amountOut);

     |  // @audit-issue Save gas by call this in assembly
 192 |  updatedInstructions.postBridge.swapPayload = ISwapper(swappers[
 193 |      instructions.postBridge.swapperId
 194 |  ]).updateSwapParams(
 195 |      newPostSwapParams,
 196 |      instructions.postBridge.swapPayload
 197 |  );

     |  // @audit-issue Save gas by call this in assembly
 212 |  address bridgeToken = bridgeAdapter.getBridgeToken(
 213 |      instructions.additionalArgs
 214 |  );

     |  // @audit-issue Save gas by call this in assembly
 216 |  IERC20(bridgeToken).approve(address(bridgeAdapter), amt2Bridge);

     |  // @audit-issue Save gas by call this in assembly
 241 |  IERC20(fees.feeToken).approve(
 242 |      address(feeCollector),
 243 |      fees.feeAmount
 244 |  );

     |  // @audit-issue Save gas by call this in assembly
 327 |  swappers[s.getId()] = swapper;

     |  // @audit-issue Save gas by call this in assembly
 336 |  bridgeAdapters[b.getId()] = bridge;
```
GitHub: [76](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L76), [78-81](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L78-L81), [90-93](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L90-L93), [95](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L95), [98](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L98), [152](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L152), [192-197](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L192-L197), [212-214](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L212-L214), [216](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L216), [241-244](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L241-L244), [327](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L327), [336](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L336)


```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Save gas by call this in assembly
  59 |  uint initBalance = IERC20(token).balanceOf(address(this));

     |  // @audit-issue Save gas by call this in assembly
  62 |  IERC20(token).approve(paymentOperator, amount);

     |  // @audit-issue Save gas by call this in assembly
  73 |  uint remainingBalance = IERC20(token).balanceOf(address(this)) -

     |  // @audit-issue Save gas by call this in assembly
  80 |  IERC20(token).transfer(refund, remainingBalance);
```
GitHub: [59](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L59), [62](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L62), [73](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L73), [80](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L80)


```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-issue Save gas by call this in assembly
  73 |  IERC20(token).transfer(owner, amount);
```
GitHub: [73](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L73)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue Save gas by call this in assembly
  57 |  router.MT_ETH_TRANSFER_WITH_PAYLOAD(),

     |  // @audit-issue Save gas by call this in assembly
 114 |  IERC20(bridgeToken).approve(address(router), amt2Bridge);

     |  // @audit-issue Save gas by call this in assembly
 145 |  IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn);
```
GitHub: [57](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L57), [114](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L114), [145](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L145)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Save gas by call this in assembly
  89 |  IERC20(bridgeToken).approve(address(router), amt2Bridge);

     |  // @audit-issue Save gas by call this in assembly
 207 |  IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn);
```
GitHub: [89](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L89), [207](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L207)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Save gas by call this in assembly
  44 |  IERC20(token).transfer(user, amount);

     |  // @audit-issue Save gas by call this in assembly
  55 |  IERC20(token).transfer(recipient, amount);

     |  // @audit-issue Save gas by call this in assembly
  91 |  IWETH(wrapped).deposit{value: swapParams.amountIn}();

     |  // @audit-issue Save gas by call this in assembly
 137 |  IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);

     |  // @audit-issue Save gas by call this in assembly
 138 |  amountOut = IV3SwapRouter(uniswap_router).exactInput(params);

     |  // @audit-issue Save gas by call this in assembly
 158 |  IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);

     |  // @audit-issue Save gas by call this in assembly
 159 |  amountIn = IV3SwapRouter(uniswap_router).exactOutput(params);
```
GitHub: [44](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L44), [55](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L55), [91](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L91), [137](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L137), [138](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L138), [158](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L158), [159](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L159)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Save gas by call this in assembly
  51 |  require(weth.balanceOf(address(this)) > amount, "not enough reserves");

     |  // @audit-issue Save gas by call this in assembly
 178 |  weth.deposit{value: _amount}();

     |  // @audit-issue Save gas by call this in assembly
 266 |  if (weth.balanceOf(address(this)) < _amount) {

     |  // @audit-issue Save gas by call this in assembly
 267 |  dcntEth.transfer(_to, _amount);

     |  // @audit-issue Save gas by call this in assembly
 273 |  weth.transfer(_to, _amount);

     |  // @audit-issue Save gas by call this in assembly
 275 |  weth.withdraw(_amount);

     |  // @audit-issue Save gas by call this in assembly
 279 |  weth.approve(address(executor), _amount);

     |  // @audit-issue Save gas by call this in assembly
 289 |  weth.withdraw(amount);

     |  // @audit-issue Save gas by call this in assembly
 298 |  weth.transfer(msg.sender, amount);

     |  // @audit-issue Save gas by call this in assembly
 308 |  weth.deposit{value: msg.value}();

     |  // @audit-issue Save gas by call this in assembly
 309 |  dcntEth.mint(address(this), msg.value);

     |  // @audit-issue Save gas by call this in assembly
 316 |  dcntEth.burn(address(this), amount);

     |  // @audit-issue Save gas by call this in assembly
 317 |  weth.withdraw(amount);

     |  // @audit-issue Save gas by call this in assembly
 326 |  dcntEth.mint(address(this), amount);

     |  // @audit-issue Save gas by call this in assembly
 333 |  dcntEth.burn(address(this), amount);

     |  // @audit-issue Save gas by call this in assembly
 334 |  weth.transfer(msg.sender, amount);
```
GitHub: [51](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L51), [178](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L178), [266](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L266), [267](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L267), [273](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L273), [275](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L275), [279](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L279), [289](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L289), [298](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L298), [308](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L308), [309](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L309), [316](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L316), [317](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L317), [326](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L326), [333](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L333), [334](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L334)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue Save gas by call this in assembly
  30 |  uint256 balanceBefore = weth.balanceOf(address(this));

     |  // @audit-issue Save gas by call this in assembly
  31 |  weth.approve(target, amount);

     |  // @audit-issue Save gas by call this in assembly
  36 |  weth.transfer(from, amount);

     |  // @audit-issue Save gas by call this in assembly
  41 |  (balanceBefore - weth.balanceOf(address(this)));

     |  // @audit-issue Save gas by call this in assembly
  44 |  weth.transfer(from, remainingAfterCall);

     |  // @audit-issue Save gas by call this in assembly
  60 |  weth.withdraw(amount);
```
GitHub: [30](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L30), [31](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L31), [36](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L36), [41](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L41), [44](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L44), [60](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L60)


## [G-21] Use hardcoded address instead of `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's `script.sol` and solmate's `LibRlp.sol` contracts can help achieve this.
Refrences:-[book.getfoundry](https://book.getfoundry.sh/reference/forge-std/compute-create-address)-[twitter](https://twitter.com/transmissions11/status/1518507047943245824).

*Instances: 26*

```solidity
File: src/UTB.sol

     |  // @audit-issue Pre-calculate & hardcode instead
  85 |  address(this),

     |  // @audit-issue Pre-calculate & hardcode instead
 238 |  address(this),
```
GitHub: [85](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L85), [238](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L238)


```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Pre-calculate & hardcode instead
  59 |  uint initBalance = IERC20(token).balanceOf(address(this));

     |  // @audit-issue Pre-calculate & hardcode instead
  61 |  IERC20(token).transferFrom(msg.sender, address(this), amount);

     |  // @audit-issue Pre-calculate & hardcode instead
  73 |  uint remainingBalance = IERC20(token).balanceOf(address(this)) -
```
GitHub: [59](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L59), [61](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L61), [73](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L73)


```solidity
File: src/UTBFeeCollector.sol

     |  // @audit-issue Pre-calculate & hardcode instead
  58 |  address(this),
```
GitHub: [58](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L58)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue Pre-calculate & hardcode instead
 111 |  address(this),

     |  // @audit-issue Pre-calculate & hardcode instead
 141 |  address(this),
```
GitHub: [111](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L111), [141](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L141)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Pre-calculate & hardcode instead
  88 |  IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);
```
GitHub: [88](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Pre-calculate & hardcode instead
  85 |  address(this),

     |  // @audit-issue Pre-calculate & hardcode instead
 132 |  recipient: address(this),

     |  // @audit-issue Pre-calculate & hardcode instead
 152 |  recipient: address(this),
```
GitHub: [85](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L85), [132](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L132), [152](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L152)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Pre-calculate & hardcode instead
  51 |  require(weth.balanceOf(address(this)) > amount, "not enough reserves");

     |  // @audit-issue Pre-calculate & hardcode instead
 181 |  weth.transferFrom(msg.sender, address(this), _amount);

     |  // @audit-issue Pre-calculate & hardcode instead
 186 |  address(this), // from address that has dcntEth (so DecentRouter)

     |  // @audit-issue Pre-calculate & hardcode instead
 266 |  if (weth.balanceOf(address(this)) < _amount) {

     |  // @audit-issue Pre-calculate & hardcode instead
 288 |  dcntEth.transferFrom(msg.sender, address(this), amount);

     |  // @audit-issue Pre-calculate & hardcode instead
 297 |  dcntEth.transferFrom(msg.sender, address(this), amount);

     |  // @audit-issue Pre-calculate & hardcode instead
 309 |  dcntEth.mint(address(this), msg.value);

     |  // @audit-issue Pre-calculate & hardcode instead
 316 |  dcntEth.burn(address(this), amount);

     |  // @audit-issue Pre-calculate & hardcode instead
 325 |  weth.transferFrom(msg.sender, address(this), amount);

     |  // @audit-issue Pre-calculate & hardcode instead
 326 |  dcntEth.mint(address(this), amount);

     |  // @audit-issue Pre-calculate & hardcode instead
 333 |  dcntEth.burn(address(this), amount);
```
GitHub: [51](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L51), [181](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L181), [186](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L186), [266](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L266), [288](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L288), [297](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L297), [309](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L309), [316](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L316), [325](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L325), [326](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L326), [333](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L333)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue Pre-calculate & hardcode instead
  30 |  uint256 balanceBefore = weth.balanceOf(address(this));

     |  // @audit-issue Pre-calculate & hardcode instead
  41 |  (balanceBefore - weth.balanceOf(address(this)));

     |  // @audit-issue Pre-calculate & hardcode instead
  75 |  weth.transferFrom(msg.sender, address(this), amount);
```
GitHub: [30](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L30), [41](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L41), [75](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L75)


## [G-22] Use named return values

Using named return values instead of explicitly calling `return` saves ~13 execution gas per call and >1000 deployment gas per instance.

*Instances: 14*

```solidity
File: src/UTB.sol

     |  // @audit-issue Provide names for all return parameters
 207 |  function approveAndCheckIfNative(
 208 |      BridgeInstructions memory instructions,
 209 |      uint256 amt2Bridge
 210 |  ) private returns (bool) {

     |  // @audit-issue Provide names for all return parameters
 259 |  function bridgeAndExecute(
 260 |      BridgeInstructions calldata instructions,
 261 |      FeeStructure calldata fees,
 262 |      bytes calldata signature
 263 |  )
 264 |      public
 265 |      payable
 266 |      retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
 267 |      returns (bytes memory)

     |  // @audit-issue Provide names for all return parameters
 282 |  function callBridge(
 283 |      uint256 amt2Bridge,
 284 |      uint bridgeFee,
 285 |      BridgeInstructions memory instructions
 286 |  ) private returns (bytes memory) {
```
GitHub: [207-210](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L207-L210), [259-268](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L259-L268), [282-286](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L282-L286)


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

     |  // @audit-issue Provide names for all return parameters
  30 |  function getId() public pure returns (uint8) {

     |  // @audit-issue Provide names for all return parameters
  67 |  function getBridgeToken(
  68 |      bytes calldata /*additionalArgs*/
  69 |  ) external view returns (address) {

     |  // @audit-issue Provide names for all return parameters
  73 |  function getBridgedAmount(
  74 |      uint256 amt2Bridge,
  75 |      address /*tokenIn*/,
  76 |      address /*tokenOut*/
  77 |  ) external pure returns (uint256) {
```
GitHub: [30](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L30), [67-69](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L67-L69), [73-77](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L73-L77)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Provide names for all return parameters
  41 |  function getId() public pure returns (uint8) {

     |  // @audit-issue Provide names for all return parameters
  61 |  function getBridgedAmount(
  62 |      uint256 amt2Bridge,
  63 |      address /*tokenIn*/,
  64 |      address /*tokenOut*/
  65 |  ) external pure returns (uint256) {

     |  // @audit-issue Provide names for all return parameters
 114 |  function getLzTxObj(
 115 |      bytes calldata additionalArgs
 116 |  ) private pure returns (IStargateRouter.lzTxObj memory) {

     |  // @audit-issue Provide names for all return parameters
 124 |  function getDstChainId(
 125 |      bytes calldata additionalArgs
 126 |  ) private pure returns (uint16) {

     |  // @audit-issue Provide names for all return parameters
 134 |  function getSrcPoolId(
 135 |      bytes calldata additionalArgs
 136 |  ) private pure returns (uint120) {

     |  // @audit-issue Provide names for all return parameters
 144 |  function getDstPoolId(
 145 |      bytes calldata additionalArgs
 146 |  ) private pure returns (uint120) {
```
GitHub: [41](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L41), [61-65](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L61-L65), [114-116](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L114-L116), [124-126](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L124-L126), [134-136](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L134-L136), [144-146](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L144-L146)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Provide names for all return parameters
  28 |  function getId() public pure returns (uint8) {

     |  // @audit-issue Provide names for all return parameters
  32 |  function updateSwapParams(
  33 |      SwapParams memory newSwapParams,
  34 |      bytes memory payload
  35 |  ) external pure returns (bytes memory) {
```
GitHub: [28](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L28), [32-35](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L32-L35)


## [G-23] Use of low-level `call()` can be optimized with assembly

Low-level calls, when optimized with assembly, can save gas by avoiding unnecessary operations related to unused returndata. In a typical `.call`, Solidity automatically allocates memory and handles returndata even if it's not used, incurring extra gas costs. By using assembly, a developer can precisely control the execution, selectively ignoring or handling returndata, thereby optimizing gas usage. This specific control over the EVM allows for more efficient execution of calls by eliminating unnecessary memory operations, providing a more gas-efficient method when unused returndata is a concern. However, such optimization should be handled with care, as improper use of assembly might lead to vulnerabilities.

*Instances: 7*

```solidity
File: src/UTBExecutor.sol

     |  // @audit-issue Consider using call in assembly for gas savings
  52 |  (success, ) = target.call{value: amount}(payload);

     |  // @audit-issue Consider using call in assembly for gas savings
  54 |  (refund.call{value: amount}(""));

     |  // @audit-issue Consider using call in assembly for gas savings
  65 |  (success, ) = target.call{value: extraNative}(payload);

     |  // @audit-issue Consider using call in assembly for gas savings
  67 |  (refund.call{value: extraNative}(""));

     |  // @audit-issue Consider using call in assembly for gas savings
  70 |  (success, ) = target.call(payload);
```
GitHub: [52](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L52), [54](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L54), [65](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L65), [67](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L67), [70](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L70)


```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

     |  // @audit-issue Consider using call in assembly for gas savings
  33 |  (bool success, ) = target.call(callPayload);

     |  // @audit-issue Consider using call in assembly for gas savings
  61 |  (bool success, ) = target.call{value: amount}(callPayload);
```
GitHub: [33](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L33), [61](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentBridgeExecutor.sol#L61)


## [G-24] Using `storage` instead of `memory` for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (**2100 gas**) for *each* field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct.

*Instances: 4*

```solidity
File: src/UTB.sol

  77 |  swapParams.tokenIn = address(wrapped);

 186 |  newPostSwapParams.amountIn = IBridgeAdapter(
 187 |      bridgeAdapters[instructions.bridgeId]
 188 |  ).getBridgedAmount(amountOut, tokenOut, newPostSwapParams.tokenIn);

 192 |  updatedInstructions.postBridge.swapPayload = ISwapper(swappers[
 193 |      instructions.postBridge.swapperId
 194 |  ]).updateSwapParams(
 195 |      newPostSwapParams,
 196 |      instructions.postBridge.swapPayload
 197 |  );
```
GitHub: [77](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L77), [186-188](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L186-L188), [192-197](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L192-L197)


```solidity
File: src/swappers/UniSwapper.sol

  90 |  swapParams.tokenIn = wrapped;
```
GitHub: [90](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L90)


## [G-25] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**

*Instances: 2*

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Switch to <x> + <y> and <x> - <y>
  56 |  balanceOf[msg.sender] += amount;

     |  // @audit-issue Switch to <x> + <y> and <x> - <y>
  64 |  balanceOf[msg.sender] -= amount;
```
GitHub: [56](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L56), [64](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L64)


## [G-26] `abi.encode()` may be less efficient than `abi.encodePacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

*Instances: 7*

```solidity
File: src/UTB.sol

     |  // @audit-issue Consider abi.encodePacked
 115 |  retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)

     |  // @audit-issue Consider abi.encodePacked
 266 |  retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
```
GitHub: [115](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L115), [266](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L266)


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

     |  // @audit-issue Consider abi.encodePacked
  81 |  bridgePayload = abi.encode(
  82 |      postBridge,
  83 |      target,
  84 |      paymentOperator,
  85 |      payload,
  86 |      refund
  87 |  );
```
GitHub: [81-87](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L81-L87)


```solidity
File: src/swappers/UniSwapper.sol

     |  // @audit-issue Consider abi.encodePacked
  40 |  return abi.encode(newSwapParams, receiver, refund);
```
GitHub: [40](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L40)


```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

     |  // @audit-issue Consider abi.encodePacked
  99 |  destBridge = bytes32(abi.encode(destinationBridges[_dstChainId]));

     |  // @audit-issue Consider abi.encodePacked
 101 |  payload = abi.encode(msgType, msg.sender, _toAddress, deliverEth);

     |  // @audit-issue Consider abi.encodePacked
 103 |  payload = abi.encode(
 104 |      msgType,
 105 |      msg.sender,
 106 |      _toAddress,
 107 |      deliverEth,
 108 |      additionalPayload
 109 |  );
```
GitHub: [99](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L99), [101](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L101), [103-109](https://github.com/code-423n4/2024-01-decent/blob/main/lib/decent-bridge/src/DecentEthRouter.sol#L103-L109)
