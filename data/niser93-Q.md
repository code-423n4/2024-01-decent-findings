## Summary

|       |Issue  |Instances|
|:-----:|:------|:-------:|
|[[NC-1](#nc-01-assembly-blocks-should-have-extensive-comments)]|Assembly blocks should have extensive comments|1|
|[[NC-2](#nc-02-complex-math-should-be-split-into-multiple-steps)]|Complex math should be split into multiple steps|2|
|[[NC-3](#nc-3-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct-for-readability)]|Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, for readability|7|
|[[NC-4](#nc-4-variables-need-not-be-initialized-to-zero)]|Variables need not be initialized to zero|4|
|[[NC-5](#nc-5-avoid-defining-useless-parameters)]|Avoid defining useless parameters|1|
|[[NC-6](#nc-6-retrievetokenin-parameter-in-performswap-is-always-true)]|`retrieveTokenIn` parameter in `performSwap()` is always true|1|
|Overall| |16|

### [NC-01] When you use assembly blocks, you should add comments
Using assembly code can can be dangerous. Moreover, it is harder to audit. Please use comments to describe assembly code behavior. In reported case, add some reference to ECDSA signature and `r`, `s`, and `v` definitions.            

```
File: UTBFeeCollector.sol

31           assembly {
32               r := mload(add(signature, 32))
33               s := mload(add(signature, 64))
34               v := byte(0, mload(add(signature, 96)))
35           }
```
[[31-35](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L31-L35)]

### [NC-02] Complex math operation
It could be better to use shorter math formula in order to have cleaner code. An alternative is to implement a function, if this kind of operation is performed multiple times.

```diff
+   function doComplexComputation(uint256 a, uint256 b, uint256 c) publiv pure{
+       return (a * (b - c)) / b;
+   }
```

*Two instances of the issue:*       
```
File: StargateBridgeAdapter.sol

66        return (amt2Bridge * (1e4 - SG_FEE_BPS)) / 1e4;
```
[[66](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L66)]


```
File: StargateBridgeAdapter.sol

176              (amt2Bridge * (10000 - SG_FEE_BPS)) / 10000, // the min qty you would accept on the destination, fee is 6 bips
```
[[176](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L176)]

### [NC-3] More than one mapping can be combined into a struct

*Seven instances of the issue:*          

```
File: DecentBridgeAdapter.sol

16       mapping(uint256 => uint16) lzIdLookup;
17       mapping(uint16 => uint256) chainIdLookup;
```
[[16](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L16), [17](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L17)]

```
File: StargateBridgeAdapter.sol

25       mapping(uint256 => address) public destinationBridgeAdapter;
26       mapping(uint256 => uint16) lzIdLookup;
27       mapping(uint16 => uint256) chainIdLookup;
```
[[25](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L25), [26](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L26), [27](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L27)]

```
File: UTB.sol

21       mapping(uint8 => address) public swappers;
22       mapping(uint8 => address) public bridgeAdapters;
```
[[21](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L21), [22](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L22)]


### [NC-4] Variables need not be initialized to zero
This operation is useless and make the code dirty.

*Four instances of the issue:*           

```
File: DecentBridgeAdapter.sol

13       uint8 public constant BRIDGE_ID = 0;
```
[[13](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L13)]

```
File: SwapParams.sol

5        uint8 constant EXACT_IN = 0;
```
[[5](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L5)]

```
File: UTB.sol

234              uint value = 0;
```
[[234](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L234)]

```
File: UniSwapper.sol

16       uint8 public constant SWAPPER_ID = 0;
```
[[16](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L16)]

### [NC-5] Avoid defining useless parameters 

```diff
File: StargateBridgeAdapter.sol

   61      function getBridgedAmount(
   62          uint256 amt2Bridge,
-  63          address /*tokenIn*/,
-  64          address /*tokenOut*/
   65      ) external pure returns (uint256) {
   66          return (amt2Bridge * (1e4 - SG_FEE_BPS)) / 1e4;
   67      }

```
[[61-67](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L61-L67)]

### [NC-6] `retrieveTokenIn` parameter in `performSwap()` is always true
                
In UTB.sol there are two `performSwap()` definition. However, this function is always called with `retrieveTokenIn=true`.

```diff
File: UTB.sol

-  48      /**
-  49       * @dev Performs a swap with a default setting to retrieve ERC20.
-  50       * @param swapInstructions The swapper ID and calldata to execute a swap.
-  51       */
-  52      function performSwap(
-  53          SwapInstructions memory swapInstructions
-  54      ) private returns (address tokenOut, uint256 amountOut) {
-  55          return performSwap(swapInstructions, true);
-  56      }
-  57  
   58      /**
   59       * @dev Performs a swap with the requested swapper and swap calldata.
   60       * @param swapInstructions The swapper ID and calldata to execute a swap.
   61       * @param retrieveTokenIn Flag indicating whether to transfer ERC20 for the swap.
   62       */
   63      function performSwap(
   64          SwapInstructions memory swapInstructions,
-  65          bool retrieveTokenIn
   66      ) private returns (address tokenOut, uint256 amountOut) {
   67          ISwapper swapper = ISwapper(swappers[swapInstructions.swapperId]);
   68  
   69          SwapParams memory swapParams = abi.decode(
   70              swapInstructions.swapPayload,
   71              (SwapParams)
   72          );
   73  
   74          if (swapParams.tokenIn == address(0)) {
   75              require(msg.value >= swapParams.amountIn, "not enough native");
   76              wrapped.deposit{value: swapParams.amountIn}();
   77              swapParams.tokenIn = address(wrapped);
   78              swapInstructions.swapPayload = swapper.updateSwapParams(
   79                  swapParams,
   80                  swapInstructions.swapPayload
   81              );
-  82          } else if (retrieveTokenIn) {
+  82          } else {
   83              IERC20(swapParams.tokenIn).transferFrom(
   84                  msg.sender,
   85                  address(this),
   86                  swapParams.amountIn
   87              );
   88          }
   89

```

[[48-89](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L48-L89)]