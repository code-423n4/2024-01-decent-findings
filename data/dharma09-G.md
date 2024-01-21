
- [Gas report](#gas-report)
  - [Table Of Contents](#table-of-contents)
  - [Refactor `execute()` function for better gas saving](#g-01-refactor-execute-function-for-better-gas-saving)
  - [Avoid the state variable read in modifier, instead we can use the memory variable](#g-02-avoid-the-state-variable-read-in-modifier-instead-we-can-use-the-memory-variable)
  - [Mark this value as constant](#g-03-mark-this-value-as-constant)
  - [storage variable should be cached with stack variable (This instance are not in Bot report )](#g-04-storage-variable-should-be-cached-with-stack-variable-this-instance-are-not-in-bot-report)
  - [No need to cache state variables only used once](#g-05-no-need-to-cache-state-variables-only-used-once)
  - [remove function and replace with inline condition or modifier](#g-06-remove-function-and-replace-with-inline-condition-or-modifier)
  - [State variables only set in the constructor should be declared immutable (this instance is not cover in bot report)](#g-07-state-variables-only-set-in-the-constructor-should-be-declared-immutable-this-instance-is-not-cover-in-bot-report)
  - [Leverage mapping and dot notation for struct assignment](#g-08-leverage-mapping-and-dot-notation-for-struct-assignment)

# Gas report

## Table Of Contents

## [G-01] Refactor `execute()` function for better gas saving

**Impact :**

In this function first if statement check `!gasCurrencyIsEth`  which is state variable than we check !`deliverEth`. 

**Proof of Code :**

So the trick is to check the first local variable and if it does not satisfy the if condition, then the else part is executed, allowing us to avoid state reading.

- [DecentBridgeExecutor.sol#L68](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L68C4-L82C6)

```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol
68: function execute(
        address from,
        address target,
        bool deliverEth,
        uint256 amount,
        bytes memory callPayload
    ) public onlyOwner {
        weth.transferFrom(msg.sender, address(this), amount);

@>        if (!gasCurrencyIsEth || !deliverEth) { //@audit change order of if condition bcz first check state variable
            _executeWeth(from, target, amount, callPayload);
        } else {
            _executeEth(from, target, amount, callPayload);
        }
    }
```

**Optimized code :** 

```diff
File: lib/decent-bridge/src/DecentBridgeExecutor.sol
68: function execute(
        address from,
        address target,
        bool deliverEth,
        uint256 amount,
        bytes memory callPayload
    ) public onlyOwner {
        weth.transferFrom(msg.sender, address(this), amount);

-        if (!gasCurrencyIsEth || !deliverEth) { 
+        if (!deliverEth || !gasCurrencyIsEth) {
            _executeWeth(from, target, amount, callPayload);
        } else {
            _executeEth(from, target, amount, callPayload);
        }
    }
```

## [G-02]  Avoid the state variable read in modifier, instead we can use the memory variable

**Impact:** In the modifier avoid the state variable read, instead use the memory variable

To prevent integer overflow/underflow issues, consider using the SafeMath library. This library provides safe arithmetic operations. If you're not already using it, you can add it to your contract.

**Proof of Concept :** 

`userIsWithdrawing` modifier update the balance of the user after withdraw, so based on below logic we can avoid state read-through using the memory value of `balanceog[msg.sender]`, and as a result we can save gas.

- [DecentEthRouter.sol#L60](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L60C3-L65C6)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol
60: modifier userIsWithdrawing(uint256 amount) {
        uint256 balance = balanceOf[msg.sender];
        require(balance >= amount, "not enough balance");
        _;
@>        balanceOf[msg.sender] -= amount; //@audit use memory value
    }
```

**Optimized code :**

```diff
File: lib/decent-bridge/src/DecentEthRouter.sol
60: modifier userIsWithdrawing(uint256 amount) {
        uint256 balance = balanceOf[msg.sender];
        require(balance >= amount, "not enough balance");
        _;
-        balanceOf[msg.sender] -= amount; 
+        balanceOf[msg.sender] = balance - amount;
    }
```

## [G-03] Mark this value as constant

**Impact :** Consider using the **`constant`** keyword to declare the **`GAS_FOR_RELAY`** variable as a constant. This helps the compiler optimize the code and makes it clear that the value is intended to be constant.

**Proof of Concept:** We can save large amounts of gas by marking the variable as a constant.

- [DecentEthRouter.sol#L96](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L96C4-L112C1)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol
80: function _getCallParams(
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
        uint256 GAS_FOR_RELAY = 100000; //@audit make constant variable
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
```

**Optimized code:**

```diff
File: lib/decent-bridge/src/DecentEthRouter.sol

+       uint256 public constant GAS_FOR_RELAY = 100000; 
80: function _getCallParams(
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
-       uint256 GAS_FOR_RELAY = 100000;
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
```

## [G-04] storage variable should be cached with stack variable (This instance are not in Bot report )

**Impact :** Caching the tokenomics in a state variable is more gas-efficient compared to making recursive calls. This approach saves 100 gas and 1 SLOAD

**Proof of Concept :** 

`feeCollector` **storage variable should be cached with stack variable.**

- [UTB.sol#L228C4](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L228C4-L252C1)

**Optimized code:**

```diff
File: src/UTB.sol
228: modifier retrieveAndCollectFees(
        FeeStructure calldata fees,
        bytes memory packedInfo,
        bytes calldata signature
    ) {
+      address _feeCollector = feeCollector;
-        if (address(feeCollector) != address(0)) { //@audit cache state variable
+        if (address(_feeCollector) != address(0))
            uint value = 0;
            if (fees.feeToken != address(0)) {
                IERC20(fees.feeToken).transferFrom(
                    msg.sender,
                    address(this),
                    fees.feeAmount
                );
                IERC20(fees.feeToken).approve(
-                    address(feeCollector),
+                    address(_feeCollector),
                    fees.feeAmount
                );
            } else {
                value = fees.feeAmount;
            }
-            feeCollector.collectFees{value: value}(fees, packedInfo, signature);
+            _feeCollector.collectFees{value: value}(fees, packedInfo, signature);
        }
        _;
    }
```

**Proof of Concept :** 

`bridgeToken` **storage variable should be cached with stack variable.**

- [DecentBridgeAdapter.sol#L81](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L81C5-L125C6)

**Optimized code:**

```diff
File: src/bridge_adapters/DecentBridgeAdapter.sol
81: function bridge(
        uint256 amt2Bridge,
        SwapInstructions memory postBridge,
        uint256 dstChainId,
        address target,
        address paymentOperator,
        bytes memory payload,
        bytes calldata additionalArgs,
        address payable refund
    ) public payable onlyUtb returns (bytes memory bridgePayload) {
        require(
            destinationBridgeAdapter[dstChainId] != address(0),
            string.concat("dst chain address not set ")
        );

        uint64 dstGas = abi.decode(additionalArgs, (uint64));

        bridgePayload = abi.encodeCall(
            this.receiveFromBridge,
            (postBridge, target, paymentOperator, payload, refund)
        );

        SwapParams memory swapParams = abi.decode(
            postBridge.swapPayload,
            (SwapParams)
        );
+       address _bridgeToken = bridgeToken;
        if (!gasIsEth) {
-            IERC20(bridgeToken).transferFrom( //@audit bridgeToken
+            IERC20(_bridgeToken).transferFrom(
                msg.sender,
                address(this),
                amt2Bridge
            );
-            IERC20(bridgeToken).approve(address(router), amt2Bridge);
+            IERC20(_bridgeToken).approve(address(router), amt2Bridge)
        }

        router.bridgeWithPayload{value: msg.value}( 
            lzIdLookup[dstChainId],
            destinationBridgeAdapter[dstChainId],
            swapParams.amountIn,
            false,
            dstGas,
            bridgePayload
        );
    }
```

## [G-05] No need to cache **state variables only used once**

**Impact :** If a state variable is only read but not modified within a function, the compiler can sometimes optimize the read operation. It may choose to read the value only once and use that value throughout the function, so there is no need to cache state variable

**Proof of Concept : I**n `getDestAdapter` function check zero address against `destinationBridgeAdapter[chainID]`.

- [StargateBridgeAdapter.sol#L153](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L153C1-L162C1)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol
154: function getDestAdapter(uint chainId) private view returns (address dstAddr) { 
        dstAddr = destinationBridgeAdapter[chainId]; //@audit no need to cache

        require(
            dstAddr != address(0),
            string.concat("dst chain address not set ")
        );
    }
```

**Optimized Code:**

```diff
File: src/bridge_adapters/StargateBridgeAdapter.sol
154: function getDestAdapter(uint chainId) private view returns (address dstAddr) { 
-        dstAddr = destinationBridgeAdapter[chainId]; //@audit no need to cache

        require(
-            dstAddr != address(0),
+            destinationBridgeAdapter[chainId] != address(0),
            string.concat("dst chain address not set ")
        );
    }
```

## [G-06] remove function and replace with inline condition or modifier

**Impact :** Modifiers are generally more efficient in terms of gas consumption because they are designed to modify the behavior of functions without duplicating code. This can lead to more concise and gas-efficient contracts.

**Proof of Concept :**

Refactor `getDestAdapter` function and implement inline require a check or make a separate modifier will check for zero address.

- [StargateBridgeAdapter.sol#L153](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L153C1-L162C1)

```solidity
File: 
154: function getDestAdapter(uint chainId) private view returns (address dstAddr) { //@audit remove this function instead we can make modifier
        dstAddr = destinationBridgeAdapter[chainId];

        require(
            dstAddr != address(0),
            string.concat("dst chain address not set ")
        );
    }
```

In this below function `getDestAdapter` used and check for zero address we can modify below function for better gas efficiency

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol
163: function callBridge(
        uint256 amt2Bridge,
        uint256 dstChainId,
        bytes memory bridgePayload,
        bytes calldata additionalArgs,
        address payable refund
    ) private {
        router.swap{value: msg.value}(
            getDstChainId(additionalArgs), //lzBridgeData._dstChainId, // send to LayerZero chainId
            getSrcPoolId(additionalArgs), //lzBridgeData._srcPoolId, // source pool id
            getDstPoolId(additionalArgs), //lzBridgeData._dstPoolId, // dst pool id
            refund, // refund adddress. extra gas (if any) is returned to this address
            amt2Bridge, // quantity to swap
            (amt2Bridge * (10000 - SG_FEE_BPS)) / 10000, // the min qty you would accept on the destination, fee is 6 bips
            getLzTxObj(additionalArgs), // additional gasLimit increase, airdrop, at address
            abi.encodePacked(getDestAdapter(dstChainId)),
            bridgePayload // bytes param, if you wish to send additional payload you can abi.encode() them here
        );
    }
```

```diff
File: src/bridge_adapters/StargateBridgeAdapter.sol
163: function callBridge( 
        uint256 amt2Bridge,
        uint256 dstChainId,
        bytes memory bridgePayload,
        bytes calldata additionalArgs,
        address payable refund
    ) private {
+          require(
+            destinationBridgeAdapter[dstChainId] != address(0),
+            string.concat("dst chain address not set ")
+        );
        router.swap{value: msg.value}(
            getDstChainId(additionalArgs), //lzBridgeData._dstChainId, // send to LayerZero chainId
            getSrcPoolId(additionalArgs), //lzBridgeData._srcPoolId, // source pool id
            getDstPoolId(additionalArgs), //lzBridgeData._dstPoolId, // dst pool id
            refund, // refund adddress. extra gas (if any) is returned to this address
            amt2Bridge, // quantity to swap
            (amt2Bridge * (10000 - SG_FEE_BPS)) / 10000, // the min qty you would accept on the destination, fee is 6 bips
            getLzTxObj(additionalArgs), // additional gasLimit increase, airdrop, at address
-            abi.encodePacked(getDestAdapter(dstChainId)),
+            abi.encodePacked(dstChainId),
            bridgePayload // bytes param, if you wish to send additional payload you can abi.encode() them here
        );
    }
```

## [G-07] State variables only set in the constructor should be declared immutable (this instance is not cover in bot report)

 **Impact :**
Avoids a Gsset(** 20000 gas**) in the constructor, and replaces the first access in each transaction(Gcoldsload - ** 2100 gas **) and each access thereafter(Gwarmacces - ** 100 gas ) with aPUSH32( 3 gas **).

**Proof of Code**
Whilestrings are not value types, and therefore cannot beimmutable / constant if not hard - coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for thestring accessors, and having a child contract override the functions with the hard - coded implementation - specific values.

- [DecentBridgeExecutor.sol#L13](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L13C7-L13C38)

```diff
File: DecentBridgeExecutor.sol#L13
-9:    IWETH weth; //@audit
+9:    IWETH public immutable weth;
    bool public gasCurrencyIsEth; // for chains that use ETH as gas currency

    constructor(address _weth, bool gasIsEth) Owned(msg.sender) {
        weth = IWETH(payable(_weth));
        gasCurrencyIsEth = gasIsEth;
    }

```

## [G-08] Leverage mapping and dot notation for struct assignment

**Impact:**
In dot notation, values are directly written to storage variable, When we use the current method in the code the compiler will allocate some memory to store the struct instance first before writing it to storage
**Proof of Code:**

- [UniSwapper.sol#L129](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L129C8-L135C16)

```solidity
File: src/swappers/UniSwapper.sol
128: function swapExactIn(
        SwapParams memory swapParams, // SwapParams is a struct
        address receiver
    ) public payable routerIsSet returns (uint256 amountOut) {
        swapParams = _receiveAndWrapIfNeeded(swapParams);

        IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter //@audit use struct notation
            .ExactInputParams({
                path: swapParams.path,
                recipient: address(this),
                amountIn: swapParams.amountIn,
                amountOutMinimum: swapParams.amountOut
            });

        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);//ok bot
        amountOut = IV3SwapRouter(uniswap_router).exactInput(params); 

        _sendToRecipient(receiver, swapParams.tokenOut, amountOut); 
    }

```
**Optimized code:**
```diff
128: function swapExactIn(
        SwapParams memory swapParams, // SwapParams is a struct
        address receiver
    ) public payable routerIsSet returns (uint256 amountOut) {
        swapParams = _receiveAndWrapIfNeeded(swapParams);
+     IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter.ExactInputParams
-       IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter //@audit use struct notation
-            .ExactInputParams({
-                path: swapParams.path,
-                recipient: address(this),
-                amountIn: swapParams.amountIn,
-                amountOutMinimum: swapParams.amountOut
-            });
+          params.path: swapParams.path,
+          params.recipient: address(this),
+          params.amountIn: swapParams.amountIn,
+          params.amountOutMinimum: swapParams.amountOut

        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);//ok bot
        amountOut = IV3SwapRouter(uniswap_router).exactInput(params); 

        _sendToRecipient(receiver, swapParams.tokenOut, amountOut); 
    }
```
 
- [UniSwapper.sol#L149](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L149C7-L156C16)
 **Proof of Code:**
 ```solidity
File: src/swappers/UniSwapper.sol
143:  function swapExactOut( 
        SwapParams memory swapParams,
        address receiver,
        address refundAddress
    ) public payable routerIsSet returns (uint256 amountIn) {
        swapParams = _receiveAndWrapIfNeeded(swapParams);
        IV3SwapRouter.ExactOutputParams memory params = IV3SwapRouter //@audit use struct notation
            .ExactOutputParams({
                path: swapParams.path,
                recipient: address(this),
                //deadline: block.timestamp, 
                amountOut: swapParams.amountOut,
                amountInMaximum: swapParams.amountIn
            });

        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn); 
        amountIn = IV3SwapRouter(uniswap_router).exactOutput(params); 

        // refund sender
        _refundUser(
            refundAddress,
            swapParams.tokenIn,
            params.amountInMaximum - amountIn
        );

        _sendToRecipient(receiver, swapParams.tokenOut, swapParams.amountOut); 
    }
 ```
 **Optimizes Code**
```diff
File: src/swappers/UniSwapper.sol
143:  function swapExactOut( 
        SwapParams memory swapParams,
        address receiver,
        address refundAddress
    ) public payable routerIsSet returns (uint256 amountIn) {
        swapParams = _receiveAndWrapIfNeeded(swapParams);
+     IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter.ExactInputParams        
-        IV3SwapRouter.ExactOutputParams memory params = IV3SwapRouter //@audit use struct notation
-            .ExactOutputParams({
-                path: swapParams.path,
-                recipient: address(this),
-                //deadline: block.timestamp, 
-                amountOut: swapParams.amountOut,
-                amountInMaximum: swapParams.amountIn
-            });
+                params.path: swapParams.path,
+                params.recipient: address(this),
+                //deadline: block.timestamp, 
+                params.amountOut: swapParams.amountOut,
+                params.amountInMaximum: swapParams.amountIn
        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn); 
        amountIn = IV3SwapRouter(uniswap_router).exactOutput(params); 

        // refund sender
        _refundUser(
            refundAddress,
            swapParams.tokenIn,
            params.amountInMaximum - amountIn
        );

        _sendToRecipient(receiver, swapParams.tokenOut, swapParams.amountOut); 
    }
```