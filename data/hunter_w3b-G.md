# Codebase Optimization Report

## Table Of Contents

- [Codebase Optimization Report](#codebase-optimization-report)

  - [Table Of Contents](#table-of-contents)
  - [\[G-01\]Using immutable on variables that are only set in the constructor and never after (Save 8400 Gas)](#g-01using-immutable-on-variables-that-are-only-set-in-the-constructor-and-never-after-save-8400-gas)
  - [\[G-02\]State variables should be cached in stack variables rather than re reading them from storage (Save 1552 Gas)](#g-02state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage-save-1552-gas)
  - [\[G-03\]Assigning STATE VARIABLES directly with named struct constructors wastes gas](#g-03assigning-state-variables-directly-with-named-struct-constructors-wastes-gas)
  - [\[G-04\]Using STORAGE instead of MEMORY for structs saves gas](#g-04using-storage-instead-of-memory-for-structs-saves-gas)
  - [\[G-05\]Using CALLDATA instead of MEMORY for read only arguments in external functions](#g-05using-calldata-instead-of-memory-for-read-only-arguments-in-external-functions)
  - [\[G-06\]Using constants directly rather than caching the value saves gas](#g-06using-constants-directly-rather-than-caching-the-value-saves-gas)
  - [\[G-07\]Avoid unnecessary public variables](#g-07avoid-unnecessary-public-variables)
  - [\[G-08\]We can save an entire SLOAD (2100 Gas) by short circuiting the operations](#g-08we-can-save-an-entire-sload-2100-gas-by-short-circuiting-the-operations)
  - [\[G-09\]Using fixed BYTES CONSTANTS is cheaper than using string](#g-09using-fixed-bytes-constants-is-cheaper-than-using-string)
  - [\[G-10\]Using assembly to revert with an error message](#g-10using-assembly-to-revert-with-an-error-message)
  - [\[G-11\]Use ASSEMBLY to hash instead of solidity](#g-11use-assembly-to-hash-instead-of-solidity)

**The bot did an amazing job on this audit,but still missed some findings**

## [G-01]Using immutable on variables that are only set in the constructor and never after (Save 8400 Gas)

**NOTE: This instance was missed by the bot.**
Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD.

- **Gas per instance**: `2.1K`
- **Total Instances**: `4`
- **Total Gas Saved**: `2.1 * 4 = 8400 Gas`

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L19

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

19    address bridgeToken;
```

```diff
-     address bridgeToken;
+     address  immutable bridgeToken;
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L34

```solidity
File: src/DecentEthRouter.sol

15    IDecentBridgeExecutor public executor;
```

```diff
File: src/DecentEthRouter.sol

-    IDecentBridgeExecutor public executor;
+    IDecentBridgeExecutor public immutable executor;
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L13

```solidity
File: src/DecentBridgeExecutor.sol

9    IWETH weth;
10    bool public gasCurrencyIsEth; // for chains that use ETH as gas currency
```

```diff
-    IWETH weth;
+    IWETH  immutable weth;

-    bool public gasCurrencyIsEth;
+    bool public immutable gasCurrencyIsEth;
```

## [G-02]State variables should be cached in stack variables rather than re reading them from storage (Save 1552 Gas)

**This instance was missed by the bot.**

- **Gas per instance**: `97`
- **Total Instances**: `16`
- **Total Gas Saved**: `97 * 16 = 1552 Gas`

```solidity
File: main/src/UTB.sol

76            wrapped.deposit{value: swapParams.amountIn}();
77            swapParams.tokenIn = address(wrapped);
98            wrapped.withdraw(amountOut);


143            executor.execute{value: amountOut}(
152            IERC20(tokenOut).approve(address(executor), amountOut);
153            executor.execute(


// @audit   cache   feeCollector  in local variable

233        if (address(feeCollector) != address(0)) {
242                    address(feeCollector),
248            feeCollector.collectFees{value: value}(fees, packedInfo, signature);
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L76

```solidity
File: src/DecentBridgeExecutor.sol

// @audit   cache   weth  in local variable
30         uint256 balanceBefore = weth.balanceOf(address(this));
41            (balanceBefore - weth.balanceOf(address(this)));
44        weth.transfer(from, remainingAfterCall);
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L30

```solidity
File: src/DecentEthRouter.sol


266        if (weth.balanceOf(address(this)) < _amount) {

273                weth.transfer(_to, _amount);

275                weth.withdraw(_amount);

279            weth.approve(address(executor), _amount);
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L266

## [G-03]Assigning STATE VARIABLES directly with named struct constructors wastes gas

Using named arguments for struct means that the compiler needs to organize the fields in memory before doing the assignment, which wastes gas.
Set each field directly in storage (use dot-notation), or use the unnamed version of the constructor.

```solidity
File: src/DecentEthRouter.sol

170        ICommonOFT.LzCallParams memory callParams = ICommonOFT.LzCallParams({
171            refundAddress: payable(msg.sender),
172            zroPaymentAddress: address(0x0),
173            adapterParams: adapterParams
174        });
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L170

```solidity
File: src/swappers/UniSwapper.

130            .ExactInputParams({
131                path: swapParams.path,
132                recipient: address(this),
133                amountIn: swapParams.amountIn,
134                amountOutMinimum: swapParams.amountOut
135            });



150            .ExactOutputParams({
151                path: swapParams.path,
152                recipient: address(this),
153                //deadline: block.timestamp,
154                amountOut: swapParams.amountOut,
155                amountInMaximum: swapParams.amountIn
156            });
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L130-L135

## [G-04]Using STORAGE instead of MEMORY for structs saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from
storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.

If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
File:  src/bridge_adapters/DecentBridgeAdapter.sol

51        SwapParams memory swapParams = abi.decode(

103        SwapParams memory swapParams = abi.decode(

134        SwapParams memory swapParams = abi.decode(

```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L51

```solidity
File: src/DecentEthRouter.sol

170        ICommonOFT.LzCallParams memory callParams = ICommonOFT.LzCallParams({
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L170

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

202        SwapParams memory swapParams = abi.decode(

```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L202

```solidity
File: src/swappers/UniSwapper.sol

129        IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter

149        IV3SwapRouter.ExactOutputParams memory params = IV3SwapRouter
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L129

```solidity
File: src/UTB.sol

69        SwapParams memory swapParams = abi.decode(

181        SwapParams memory newPostSwapParams = abi.decode(

```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L69

## [G-05]Using CALLDATA instead of MEMORY for read only arguments in external functions

**NOTE: This instance was missed by the bot.**

When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas `(i.e. 60 * <mem_array>.length)`. Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.

```solidity
File: src/DecentBridgeExecutor.sol

73        bytes memory callPayload
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L73

```solidity
File: src/DecentEthRouter.sol

120        bytes memory payload


203        bytes memory additionalPayload

243        bytes memory _payload

```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L120

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

71        SwapInstructions memory postBridge,

75        bytes memory payload,

189        bytes memory payload

```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L71

```solidity
File: src/swappers/UniSwapper.sol

33        SwapParams memory newSwapParams,

34        bytes memory payload

59        bytes memory swapPayload

```

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L33

```solidity
File: src/UTB.sol

312        SwapInstructions memory postBridge,

315        bytes memory payload,
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L312

```solidity
File: src/UTBExecutor.sol


22        bytes memory payload,

44        bytes memory payload,
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L22

```solidity
File: src/UTBFeeCollector.sol

46        bytes memory packedInfo,

47        bytes memory signature
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L46

## [G-06]Using constants directly, rather than caching the value, saves gas

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

30    function getId() public pure returns (uint8) {
31        return BRIDGE_ID;
32    }

```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L30

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

41    function getId() public pure returns (uint8) {
42        return BRIDGE_ID;
43    }
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L41

```solidity
File: src/swappers/UniSwapper.sol

28    function getId() public pure returns (uint8) {
29        return SWAPPER_ID;
30    }
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L28-L30

## [G-07]Avoid unnecessary public variables

A public storage variable has an implicit public function of the same name. A public function increases the size of the jump table and adds bytecode to read the variable in question. That makes the contract larger.

```soldity
File:/home/x0w3r/progress-ON/Decent/scope/BaseAdapter.sol

7    address public bridgeExecutor;
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L7

```soldity
File: /home/x0w3r/progress-ON/Decent/scope/DcntEth.sol

6    address public router;
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L6

```soldity
File: /home/x0w3r/progress-ON/Decent/scope/DecentBridgeAdapter.sol

15    IDecentEthRouter public router;
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L15

```soldity
File: /home/x0w3r/progress-ON/Decent/scope/DecentBridgeExecutor.sol

10    bool public gasCurrencyIsEth; // for chains that use ETH as gas currency
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L10

```soldity
File: /home/x0w3r/progress-ON/Decent/scope/DecentEthRouter.sol

13    IWETH public weth;
14    IDcntEth public dcntEth;
15    IDecentBridgeExecutor public executor;
22    bool public gasCurrencyIsEth; // for chains that use ETH as gas currency
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L13

```soldity
File:/home/x0w3r/progress-ON/Decent/scope/StargateBridgeAdapter.sol

24    address public stargateEth;
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L24

```soldity
File: /home/x0w3r/progress-ON/Decent/scope/UniSwapper.sol

17    address public uniswap_router;
18    address payable public wrapped;
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L17

## [G-08]We can save an entire SLOAD (2100 Gas) by short circuiting the operations

- **Gas per instance**: `2100`
- **Total Instances**: `2`
- **Total Gas Saved**: `2 * 2100 = 4200 Gas`

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L77

```solidity
File: src/DecentBridgeExecutor.sol

77        if (!gasCurrencyIsEth || !deliverEth) {
```

The if statement has two checks `!gasCurrencyIsEth` and `!deliverEth` where the first check involves making a state read while the second check only compares a variable to a function parameter.

According to the rules of short circuit, if the first check is true, we do not have to do the second check thus in this case, we should make sure the first check is the cheapest to do.

By reordering as shown below, we can avoid making the state read if !gasCurrencyIsEth which would save us ~`2100` Gas.

```diff

-        if (!gasCurrencyIsEth || !deliverEth) {
+        if (!deliverEth || !gasCurrencyIsEth ) {
```

```solidity
File: src/DecentEthRouter.sol

272            if (!gasCurrencyIsEth || !deliverEth) {
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L272

## [G-09]Using fixed BYTES CONSTANTS is cheaper than using string

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

```solidity
File: main/src/UTBFeeCollector.sol

10    string constant BANNER = "\x19Ethereum Signed Message:\n32";
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L10

## [G-10]Using assembly to revert with an error message

When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

```solidity
File: src/bridge_adapters/BaseAdapter.sol

12        require(
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L12

```solidity
File: src/DcntEth.sol

9        require(msg.sender == router);
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L9

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

91        require(
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L19

```solidity
File: src/DecentEthRouter.sol

38        require(gasCurrencyIsEth, "Gas currency is not ETH");

43        require(

51        require(weth.balanceOf(address(this)) > amount, "not enough reserves");

62        require(balance >= amount, "not enough balance");
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L38

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

157        require(
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L81

```solidity
File: src/swappers/UniSwapper.sol

96        require(uniswap_router != address(0), "router not set");
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L137

```solidity
File: src/UTB.sol

75            require(msg.value >= swapParams.amountIn, "not enough native");
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L76

```solidity
File: src/UTBFeeCollector.sol

29        require(signature.length == 65, "Invalid signature length");

54        require(recovered == signer, "Wrong signature");
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L10

## [G-11]Use ASSEMBLY to hash instead of solidity

Use ASSEMBLY for small keccak256 hashes, in order to save gas.If the arguments to the encode call can fit into the scratch space (two words or fewer),then it's more efficient to use assembly to generate the hash `(80 gas)`

keccak256(abi.encodePacked(x, y)) -> assembly {mstore(0x00, a); mstore(0x20, b); let hash := keccak256(0x00, 0x40); }

```solidity
File: src/UTBFeeCollector.sol

49        bytes32 constructedHash = keccak256(
50            abi.encodePacked(BANNER, keccak256(packedInfo))
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L49
