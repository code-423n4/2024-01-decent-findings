# Codebase Optimization Report

## Table of Contents

- [Codebase Optimization Report](#codebase-optimization-report)
  - [Table of Contents](#table-of-contents)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Using immutable on variables that are only set in the constructor and never after (Save 8400 Gas) - Missed by bots](#using-immutable-on-variables-that-are-only-set-in-the-constructor-and-never-after-save-8400-gas---missed-by-bots)
    - [`bridgeToken` should be immutable](#bridgetoken-should-be-immutable)
    - [`executor` should be immutable](#executor-should-be-immutable)
    - [`weth` and `gasCurrencyIsEth` should be immutable](#weth-and-gascurrencyiseth-should-be-immutable)
  - [Unnecessary SLOADs due to how functions are executed](#unnecessary-sloads-due-to-how-functions-are-executed)
  - [Save an entire sload by using short circuiting to our advantage](#save-an-entire-sload-by-using-short-circuiting-to-our-advantage)
  - [Use the memory variable instead of reading the state variable](#use-the-memory-variable-instead-of-reading-the-state-variable)
  - [Unnecessary use of  concat function](#unnecessary-use-of--concat-function)
    - [No need to use concat since we only have one string](#no-need-to-use-concat-since-we-only-have-one-string)
  - [Declare a constant variable for `GAS_FOR_RELAY`](#declare-a-constant-variable-for-gas_for_relay)
  - [Refactor the modifier to save 1 SLOAD(100 Gas)](#refactor-the-modifier-to-save-1-sload100-gas)
  - [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)
  - [Cache storage values in memory to minimize SLOADs](#cache-storage-values-in-memory-to-minimize-sloads)
    - [router should be cached(saves 1 SLOAD ~100 Gas) - not found by bot](#router-should-be-cachedsaves-1-sload-100-gas---not-found-by-bot)
  - [Unused private functions should be deleted](#unused-private-functions-should-be-deleted)
  - [Conclusion](#conclusion)


## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

**Note: some titles may be similar to the bot but the issues identified were missed by bots**

## Using immutable on variables that are only set in the constructor and never after (Save 8400 Gas) - Missed by bots

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD.
Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas)

**Total Instances: 4**
**Gas per instance: 2100**
**Total Saved: 8400 Gas**

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/DecentBridgeAdapter.sol#L19

### `bridgeToken` should be immutable

```solidity
File: /src/bridge_adapters/DecentBridgeAdapter.sol
19:    address bridgeToken;
```

```diff
diff --git a/src/bridge_adapters/DecentBridgeAdapter.sol b/src/bridge_adapters/DecentBridgeAdapter.sol
index 69bf19e..daaf4f8 100644
--- a/src/bridge_adapters/DecentBridgeAdapter.sol
+++ b/src/bridge_adapters/DecentBridgeAdapter.sol
@@ -16,7 +16,7 @@ contract DecentBridgeAdapter is BaseAdapter, IBridgeAdapter {
     mapping(uint256 => uint16) lzIdLookup;
     mapping(uint16 => uint256) chainIdLookup;
     bool gasIsEth;
-    address bridgeToken;
+    address immutable bridgeToken;
```

### `executor` should be immutable

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L15
```solidity
File: /src/DecentEthRouter.sol
15:    IDecentBridgeExecutor public executor;
```

### `weth` and `gasCurrencyIsEth` should be immutable

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L9-L10
```solidity
File: /src/DecentBridgeExecutor.sol
9:    IWETH weth;
10:    bool public gasCurrencyIsEth; // for chains that use ETH as gas currency
```

## Unnecessary SLOADs due to how functions are executed

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTB.sol#L282-L301
```solidity
File: /src/UTB.sol
282:    function callBridge(
283:        uint256 amt2Bridge,
284:        uint bridgeFee,
285:        BridgeInstructions memory instructions
286:    ) private returns (bytes memory) {
287:        bool native = approveAndCheckIfNative(instructions, amt2Bridge);
288:        return
289:            IBridgeAdapter(bridgeAdapters[instructions.bridgeId]).bridge{
290:                value: bridgeFee + (native ? amt2Bridge : 0)
291:            }(
292:                amt2Bridge,
293:                instructions.postBridge,
294:                instructions.dstChainId,
295:                instructions.target,
296:                instructions.paymentOperator,
297:                instructions.payload,
298:                instructions.additionalArgs,
299:                instructions.refund
300:            );
301:    }
```
The above function makes a call to function `approveAndCheckIfNative` whose implementation is as follows
https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTB.sol#L207-L220
```solidity
    function approveAndCheckIfNative(
        BridgeInstructions memory instructions,
        uint256 amt2Bridge
    ) private returns (bool) {
        IBridgeAdapter bridgeAdapter = IBridgeAdapter(bridgeAdapters[instructions.bridgeId]);
        address bridgeToken = bridgeAdapter.getBridgeToken(
            instructions.additionalArgs
        );
        if (bridgeToken != address(0)) {
            IERC20(bridgeToken).approve(address(bridgeAdapter), amt2Bridge);
            return false;
        }
        return true;
    }
```

Now, note that in both functions we are reading the state variable  `IBridgeAdapter(bridgeAdapters[instructions.bridgeId])`

If we in line the functionality of `approveAndCheckIfNative` inside the function `callBridge` we can avoid making this unnecessary sloads

```diff
diff --git a/src/UTB.sol b/src/UTB.sol
index 3d05b38..c864b81 100644
--- a/src/UTB.sol
+++ b/src/UTB.sol
@@ -284,9 +284,20 @@ contract UTB is Owned {
         uint bridgeFee,
         BridgeInstructions memory instructions
     ) private returns (bytes memory) {
-        bool native = approveAndCheckIfNative(instructions, amt2Bridge);
+
+        bool native;
+        IBridgeAdapter bridgeAdapter = IBridgeAdapter(bridgeAdapters[instructions.bridgeId]);
+        address bridgeToken = bridgeAdapter.getBridgeToken(
+            instructions.additionalArgs
+        );
+        if (bridgeToken != address(0)) {
+            IERC20(bridgeToken).approve(address(bridgeAdapter), amt2Bridge);
+            native = false;
+        }
+        native = true;
+
         return
-            IBridgeAdapter(bridgeAdapters[instructions.bridgeId]).bridge{
+            bridgeAdapter.bridge{
                 value: bridgeFee + (native ? amt2Bridge : 0)
             }(
                 amt2Bridge,
```

The function `approveAndCheckIfNative` is no longer needed and we can simply delete the code

## Save an entire sload by using short circuiting to our advantage

**Note, if we implement the immutable finding, then we do not have to do this**
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L68-L82
```solidity
File: /src/DecentBridgeExecutor.sol
68:    function execute(
69:        address from,
70:        address target,
71:        bool deliverEth,
72:        uint256 amount,
73:        bytes memory callPayload
74:    ) public onlyOwner {
75:        weth.transferFrom(msg.sender, address(this), amount);

77:        if (!gasCurrencyIsEth || !deliverEth) {
78:            _executeWeth(from, target, amount, callPayload);
79:        } else {
80:            _executeEth(from, target, amount, callPayload);
81:        }
82:    }
```

The if condition checks the status of `gasCurrencyIsEth` which is a state variable(SLOAD) then depending on the result, it will check `deliverEth` which is a function parameter(mload)
The second check is way cheaper and thus we can use the rules of short circuiting to our advantage

```diff
diff --git a/src/DecentBridgeExecutor.sol b/src/DecentBridgeExecutor.sol
index 1a47cb5..f33e301 100644
--- a/src/DecentBridgeExecutor.sol
+++ b/src/DecentBridgeExecutor.sol
@@ -74,12 +74,12 @@ contract DecentBridgeExecutor is IDecentBridgeExecutor, Owned {
     ) public onlyOwner {
         weth.transferFrom(msg.sender, address(this), amount);

-        if (!gasCurrencyIsEth || !deliverEth) {
+        if (!deliverEth || !gasCurrencyIsEth ) {
             _executeWeth(from, target, amount, callPayload);
         } else {
             _executeEth(from, target, amount, callPayload);
         }
-    }
```

## Use the memory variable instead of reading the state variable

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/swappers/UniSwapper.sol#L79-L93
```solidity
File: /src/swappers/UniSwapper.sol
79:    function _receiveAndWrapIfNeeded(
80:        SwapParams memory swapParams
81:    ) private returns (SwapParams memory _swapParams) {
82:        if (swapParams.tokenIn != address(0)) {
83:            IERC20(swapParams.tokenIn).transferFrom(
84:                msg.sender,
85:                address(this),
86:                swapParams.amountIn
87:            );
88:            return swapParams;
89:        }
90:        swapParams.tokenIn = wrapped;
91:        IWETH(wrapped).deposit{value: swapParams.amountIn}();
92:        return swapParams;
93:    }
```

```diff
diff --git a/src/swappers/UniSwapper.sol b/src/swappers/UniSwapper.sol
index 929f710..c47e998 100644
--- a/src/swappers/UniSwapper.sol
+++ b/src/swappers/UniSwapper.sol
@@ -88,7 +88,7 @@ contract UniSwapper is UTBOwned, ISwapper {
             return swapParams;
         }
         swapParams.tokenIn = wrapped;
-        IWETH(wrapped).deposit{value: swapParams.amountIn}();
+        IWETH(swapParams.tokenIn).deposit{value: swapParams.amountIn}();
         return swapParams;
     }

```

## Unnecessary use of  concat function

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/StargateBridgeAdapter.sol#L154-L161
```solidity
File: /src/bridge_adapters/StargateBridgeAdapter.sol
154:    function getDestAdapter(uint chainId) private view returns (address dstAddr) {
155:        dstAddr = destinationBridgeAdapter[chainId];

157:        require(
158:            dstAddr != address(0),
159:            string.concat("dst chain address not set ")
160:        );
161:    }
```

```diff
diff --git a/src/bridge_adapters/StargateBridgeAdapter.sol b/src/bridge_adapters/StargateBridgeAdapter.sol
index c0339aa..b6b06fd 100644
--- a/src/bridge_adapters/StargateBridgeAdapter.sol
+++ b/src/bridge_adapters/StargateBridgeAdapter.sol
@@ -155,8 +155,7 @@ contract StargateBridgeAdapter is
         dstAddr = destinationBridgeAdapter[chainId];

         require(
-            dstAddr != address(0),
-            string.concat("dst chain address not set ")
+            dstAddr != address(0),"dst chain address not set "
         );
     }
```

### No need to use concat since we only have one string

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/DecentBridgeAdapter.sol#L81-L94
```solidity
File: /src/bridge_adapters/DecentBridgeAdapter.sol
81:    function bridge(
82:        uint256 amt2Bridge,
83:        SwapInstructions memory postBridge,
84:        uint256 dstChainId,
85:        address target,
86:        address paymentOperator,
87:        bytes memory payload,
88:        bytes calldata additionalArgs,
89:        address payable refund
90:    ) public payable onlyUtb returns (bytes memory bridgePayload) {
91:        require(
92:            destinationBridgeAdapter[dstChainId] != address(0),
93:            string.concat("dst chain address not set ")
94:        );
```


```diff
diff --git a/src/bridge_adapters/DecentBridgeAdapter.sol b/src/bridge_adapters/DecentBridgeAdapter.sol
index 69bf19e..b32abe3 100644
--- a/src/bridge_adapters/DecentBridgeAdapter.sol
+++ b/src/bridge_adapters/DecentBridgeAdapter.sol
@@ -90,7 +90,7 @@ contract DecentBridgeAdapter is BaseAdapter, IBridgeAdapter {
     ) public payable onlyUtb returns (bytes memory bridgePayload) {
         require(
             destinationBridgeAdapter[dstChainId] != address(0),
-            string.concat("dst chain address not set ")
+           "dst chain address not set "
         );
```

## Declare a constant variable for `GAS_FOR_RELAY`

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L96-L97
```solidity
File: /src/DecentEthRouter.sol
96:        uint256 GAS_FOR_RELAY = 100000;
97:        uint256 gasAmount = GAS_FOR_RELAY + _dstGasForCall;
```

```diff
@@ -18,8 +18,9 @@ contract DecentEthRouter is IDecentEthRouter, IOFTReceiverV2, Owned {
     uint8 public constant MT_ETH_TRANSFER_WITH_PAYLOAD = 1;

     uint16 public constant PT_SEND_AND_CALL = 1;
+    uint256 public constant GAS_FOR_RELAY = 100000;
@@ -93,7 +94,7 @@ contract DecentEthRouter is IDecentEthRouter, IOFTReceiverV2, Owned {
             bytes memory payload
         )
     {
-        uint256 GAS_FOR_RELAY = 100000;
+
         uint256 gasAmount = GAS_FOR_RELAY + _dstGasForCall;
         adapterParams = abi.encodePacked(PT_SEND_AND_CALL, gasAmount);
         destBridge = bytes32(abi.encode(destinationBridges[_dstChainId]));
```


## Refactor the modifier to save 1 SLOAD(100 Gas)

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L60-L65
```solidity
File: /src/DecentEthRouter.sol
60:    modifier userIsWithdrawing(uint256 amount) {
61:        uint256 balance = balanceOf[msg.sender];
62:        require(balance >= amount, "not enough balance");
63:        _;
64:        balanceOf[msg.sender] -= amount;
65:    }
```
We are caching the value of `balanceOf[msg.sender]` on line 61 but due to how we perform the arithmetic operation on line 64, we cannot take advantage of the cached variable(using the short form `-=` utilizes the first variable). We can change this to the long form (`x = x - y`) which would allow us to use the variable `balance` which saves us an entire SLOAD(100 Gas)

```diff
         uint256 balance = balanceOf[msg.sender];
         require(balance >= amount, "not enough balance");
         _;
-        balanceOf[msg.sender] -= amount;
+        balanceOf[msg.sender] = balance - amount;
     }
```


## Using unchecked blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
[see resource](https://github.com/ethereum/solidity/issues/10695)

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L60-L65
```solidity
File: /src/DecentEthRouter.sol
60:    modifier userIsWithdrawing(uint256 amount) {
61:        uint256 balance = balanceOf[msg.sender];
62:        require(balance >= amount, "not enough balance");
63:        _;
64:        balanceOf[msg.sender] -= amount;
65:    }
```
The check on line 62 ensures that the arithmetic operation on line 64 would never overflow since it would only be performed if `balanceOf[msg.sender]` is greater than `amount`
```diff
@@ -61,7 +61,10 @@ contract DecentEthRouter is IDecentEthRouter, IOFTReceiverV2, Owned {
         uint256 balance = balanceOf[msg.sender];
         require(balance >= amount, "not enough balance");
         _;
-        balanceOf[msg.sender] -= amount;
+        unchecked{
+            balanceOf[msg.sender] -= amount;
+        }
+
     }

```

## Cache storage values in memory to minimize SLOADs

The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

###  router should be cached(saves 1 SLOAD ~100 Gas) - not found by bot

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/DecentBridgeAdapter.sol#L44-L65
```solidity
File: /src/bridge_adapters/DecentBridgeAdapter.sol
55:        return
56:            router.estimateSendAndCallFee(
57:                router.MT_ETH_TRANSFER_WITH_PAYLOAD(),
58:                lzIdLookup[dstChainId],
59:                target,
60:                swapParams.amountIn,
61:                dstGas,
62:                false,
63:                payload
64:            );
```

## Unused private functions should be deleted

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/StargateBridgeAdapter.sol#L100-L112
```solidity
File: /src/bridge_adapters/StargateBridgeAdapter.sol
100:    function getValue(
101:        bytes calldata additionalArgs,
102:        uint256 amt2Bridge
103:    ) private view returns (uint value) {
104:        (address bridgeToken, LzBridgeData memory lzBridgeData) = abi.decode(
105:            additionalArgs,
106:            (address, LzBridgeData)
107:        );
108:        return
109:            bridgeToken == stargateEth
110:                ? (lzBridgeData.fee + amt2Bridge)
111:                : lzBridgeData.fee;
112:    }
```


## Conclusion

It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.
