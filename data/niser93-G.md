|       |Issue  |Instances|
|:-----:|:------|:-------:|
|[[GO-01](#go-01-using-of-positive-conditional-paths)]|Using of positive conditional paths|4|
|[[GO-02](#go-02-move-the-address0-check-from-retrieveandcollectfees-to-setfeecollector-in-utbsol)]|Move the `address(0)` check from `retrieveAndCollectFees()` to `setFeeCollector()` in UTB.sol|1|
|[[GO-03](#go-03-avoid-defining-retrievetokenin-parameter-in-performswap-function-in-utbsol)]|Avoid defining `retrieveTokenIn` parameter in `performSwap()` function in UTB.sol|1|
|[[GO-04](#go-04-avoid-defining-useless-return-parameter-in-swapandmodifypostbridge-in-utbsol)]|Avoid defining useless return parameter in `swapAndModifyPostBridge` in UTB.sol|1|
|[[GO-05](#go-05-avoid-defining-local-variable-that-are-used-once)]|Avoid defining local variable that are used once|2|
|[[GO-06](#go-06-changing-condition-path-to-use--or--instead-of-)]|Changing condition path to use `>=` or `==` instead of `>`|1|
|[[GO-07](#go-07-move-important-checks-at-the-beginning-of-function)]|Move important checks at the beginning of function|1|
|[[GO-08](#go-08-modifying-gasiseth-in-gasisnoteth-to-save-not-opcode)]|Modifying `gasIsEth` in `gasIsNotEth` to save `NOT` opcode|1|
|[[GO-09](#go-09-avoid-defining-useless-parameters-in-getbridgedamount-function-in-stargatebridgeadaptersol)]|Avoid defining useless parameters in `getBridgedAmount()` function in StargateBridgeAdapter.sol|1|
|[[GO-10](#go-10-use-alternative-formulation-in-order-to-avoid-not-using-and-instead-of-or-or-vice-versa)]|Use alternative formulation in order to avoid NOT using AND instead of OR or vice versa|2|
|                 |                     |          |      |
| OVERALL              |                | 15 |
                
### [GO-01] Using of positive conditional paths
                
Many times, a different conditional path could lead to saving gas.

For example, it could permit to use `>=` instead of `>` or avoid using `NOT` operator.

*Issue's instances:* 4

#### File: [UTB.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol)

*Instances:* 1    


```diff
File: UTB.sol

Current Code:
-  215          if (bridgeToken != address(0)) {
-  216              IERC20(bridgeToken).approve(address(bridgeAdapter), amt2Bridge);
-  217              return false;
-  218          }
-  219          return true;

New Code:
+  215  		if (bridgeToken == address(0)) {
+  216              return true;
+  217          }
+  218          IERC20(bridgeToken).approve(address(bridgeAdapter), amt2Bridge);
+  219          return false;

```

*Code link:* [[215-219](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L215-L219)]



***
&nbsp;

#### File: [UTB.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol)

*Instances:* 1    


```diff
File: UTB.sol

Current Code:
-  235              if (fees.feeToken != address(0)) {
-  236                  IERC20(fees.feeToken).transferFrom(
-  237                      msg.sender,
-  238                      address(this),
-  239                      fees.feeAmount
-  240                  );
-  241                  IERC20(fees.feeToken).approve(
-  242                      address(feeCollector),
-  243                      fees.feeAmount
-  244                  );
-  245              } else {
-  246                  value = fees.feeAmount;
-  247              }

New Code:
+  235  			if (fees.feeToken == address(0)) {
+  236                  value = fees.feeAmount;
+  237              } else {
+  238                  IERC20(fees.feeToken).transferFrom(
+  239                      msg.sender,
+  240                      address(this),
+  241                      fees.feeAmount
+  242                  );
+  243                  IERC20(fees.feeToken).approve(
+  244                      address(feeCollector),
+  245                      fees.feeAmount
+  246                  );
+  247              }

```

*Code link:* [[235-247](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L235-L247)]



***
&nbsp;

#### File: [UTBExecutor.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol)

*Instances:* 1    


```diff
File: UTBExecutor.sol

Current Code:
-  53              if (!success) {
-  54                  (refund.call{value: amount}(""));
-  55              }
-  56              return;

New Code:
+  53              if (success) {
+  54                  return;
+  55              }
+  56              (refund.call{value: amount}(""));

```

*Code link:* [[53-56](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L53-L56)]



***
&nbsp;

#### File: [DecentBridgeExecutor.sol](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol)

*Instances:* 1    


```diff
File: DecentBridgeExecutor.sol

Current Code:
-  35          if (!success) {
-  36              weth.transfer(from, amount);
-  37              return;
-  38          }
-  39  
-  40          uint256 remainingAfterCall = amount -
-  41              (balanceBefore - weth.balanceOf(address(this)));
-  42  
-  43          // refund the sender with excess WETH
-  44          weth.transfer(from, remainingAfterCall);

New Code:
+  35          if (success) {
+  36              uint256 remainingAfterCall = amount -
+  37                  (balanceBefore - weth.balanceOf(address(this)));
+  38  
+  39              // refund the sender with excess WETH
+  40              weth.transfer(from, remainingAfterCall);
+  41              return;
+  42          }
+  43          weth.transfer(from, amount);

```

*Code link:* [[35-44](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L35-L44)]



***
&nbsp;

                
### [GO-02] Move the `address(0)` check from `retrieveAndCollectFees()` to `setFeeCollector()` in UTB.sol
                
The `address(0)` check inside `retrieveAndCollectFees()` function wastes gas because it verifies the address of `feeCollector` during every execute action. Set initial `feeCollector` using contructor and add a check inside `setFeeCollector` to avoid to assign `address(0)` to `feeCollector`. Moreover, if `feeCollector` is `address(0)`, `IERC20.approve()` reverts.

*Issue's instances:* 1

#### File: [UTB.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol)

*Instances:* 1    


```diff
File: UTB.sol

   40      /**
   41       * @dev Sets the fee collector.
   42       * @param _feeCollector The address of the fee collector.
   43       */
   44      function setFeeCollector(address payable _feeCollector) public onlyOwner {
+              require(_feeCollector!=address(0));
   45          feeCollector = IUTBFeeCollector(_feeCollector);
   46      }

[...]

-  233          if (address(feeCollector) != address(0)) {
   234              uint value = 0;
   235              if (fees.feeToken != address(0)) {
   236                  IERC20(fees.feeToken).transferFrom(
   237                      msg.sender,
   238                      address(this),
   239                      fees.feeAmount
   240                  );
   241                  IERC20(fees.feeToken).approve(
   242                      address(feeCollector),
   243                      fees.feeAmount
   244                  );
   245              } else {
   246                  value = fees.feeAmount;
   247              }
   248              feeCollector.collectFees{value: value}(fees, packedInfo, signature);
-  249          }

```

*Code link:* [[233-249](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L233-L249)]



***
&nbsp;

                
### [GO-03] Avoid defining `retrieveTokenIn` parameter in `performSwap()` function in UTB.sol
                
In UTB.sol there are two `performSwap()` definition. However, this function is always called with `retrieveTokenIn=true`.

*Issue's instances:* 1

#### File: [UTB.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol)

*Instances:* 1    


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

*Code link:* [[48-100](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L48-L100)]



***
&nbsp;

                
### [GO-04] Avoid defining useless return parameter in `swapAndModifyPostBridge` in UTB.sol
                


*Issue's instances:* 1

#### File: [UTB.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol)

*Instances:* 1    


```diff
File: UTB.sol
   168      function swapAndModifyPostBridge(
   169          BridgeInstructions memory instructions
   170      )
   171          private
   172          returns (
-  173              uint256 amount2Bridge,
-  174              BridgeInstructions memory updatedInstructions
+                   uint256,
+                   BridgeInstructions memory
   175          )
   176      {
   177          (address tokenOut, uint256 amountOut) = performSwap(
   178              instructions.preBridge
   179          );
   180  
   181          SwapParams memory newPostSwapParams = abi.decode(
   182              instructions.postBridge.swapPayload,
   183              (SwapParams)
   184          );
   185  
   186          newPostSwapParams.amountIn = IBridgeAdapter(
   187              bridgeAdapters[instructions.bridgeId]
   188          ).getBridgedAmount(amountOut, tokenOut, newPostSwapParams.tokenIn);
   189  
-  190          updatedInstructions = instructions;
+               BridgeInstructions memory updatedInstructions = instructions;
   191  
   192          updatedInstructions.postBridge.swapPayload = ISwapper(swappers[
   193              instructions.postBridge.swapperId
   194          ]).updateSwapParams(
   195              newPostSwapParams,
   196              instructions.postBridge.swapPayload
   197          );
   198  
-  199          amount2Bridge = amountOut;
+               return(amountOut, updatedInstructions);
   200      }

```

*Code link:* [[168-200](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L168-L200)]



***
&nbsp;

                
### [GO-05] Avoid defining local variable that are used once
                
Avoid defining a local variable can save gas. When code don't lose clarity, it could be better to write inline code.

*Issue's instances:* 2

#### File: [UTB.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol)

*Instances:* 1    


```diff
File: UTB.sol

   325      function registerSwapper(address swapper) public onlyOwner {
-  326          ISwapper s = ISwapper(swapper);
-  327          swappers[s.getId()] = swapper;
+               swappers[ISwapper(swapper).getId()] = swapper;
   328      }

```

*Code link:* [[325-328](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L325-L328)]



***
&nbsp;

#### File: [UTB.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol)

*Instances:* 1    


```diff
File: UTB.sol

   334      function registerBridge(address bridge) public onlyOwner {
-  335          IBridgeAdapter b = IBridgeAdapter(bridge);
-  336          bridgeAdapters[b.getId()] = bridge;
+               bridgeAdapters[IBridgeAdapter(bridge).getId()] = bridge;
   337      }

```

*Code link:* [[334-337](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L334-L337)]



***
&nbsp;

                
### [GO-06] Changing condition path to use `>=` or `==` instead of `>` 
                


*Issue's instances:* 1

#### File: [UTBExecutor.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol)

*Instances:* 1    


```diff
File: UTBExecutor.sol

Current Code:
-  64          if (extraNative > 0) {
-  65              (success, ) = target.call{value: extraNative}(payload);
-  66              if (!success) {
-  67                  (refund.call{value: extraNative}(""));
-  68              }
-  69          } else {
-  70              (success, ) = target.call(payload);
-  71          }

New Code:
+  64          if (extraNative == 0) {
+  65              (success, ) = target.call(payload);
+  66          } else {
+  67              (success, ) = target.call{value: extraNative}(payload);
+  68              if (!success) {
+  69                  (refund.call{value: extraNative}(""));
+  70              }
+  71          }

```

*Code link:* [[64-71](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L64-L71)]



***
&nbsp;

                
### [GO-07] Move important checks at the beginning of function
                


*Issue's instances:* 1

#### File: [UTBFeeCollector.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol)

*Instances:* 1    


```diff
File: UTBFeeCollector.sol

   44      function collectFees(
   45          FeeStructure calldata fees,
   46          bytes memory packedInfo,
   47          bytes memory signature
   48      ) public payable onlyUtb {
+              if (fees.feeToken != address(0)) return;
   49          bytes32 constructedHash = keccak256(
   50              abi.encodePacked(BANNER, keccak256(packedInfo))
   51          );
   52          (bytes32 r, bytes32 s, uint8 v) = splitSignature(signature);
   53          address recovered = ecrecover(constructedHash, v, r, s);
   54          require(recovered == signer, "Wrong signature");
-  55          if (fees.feeToken != address(0)) {
   56              IERC20(fees.feeToken).transferFrom(
   57                  utb,
   58                  address(this),
   59                  fees.feeAmount
   60              );
-  61          }
   62      }

```

*Code link:* [[44-62](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L44-L62)]



***
&nbsp;

                
### [GO-08] Modifying `gasIsEth` in `gasIsNotEth` to save `NOT` opcode
                
`gasIsEth` is used only with negation

*Issue's instances:* 1

#### File: [DecentBridgeAdapter.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol)

*Instances:* 1    


```diff
File: DecentBridgeAdapter.sol

-  18      bool gasIsEth;
+  18      bool gasIsNotEth;
   19      address bridgeToken;
   20  
   21      constructor(bool _gasIsEth, address _bridgeToken) BaseAdapter() {
-  22          gasIsEth = _gasIsEth;
+              gasIsNotEth = !_gasIsEth;
   23          bridgeToken = _bridgeToken;
   24      }

[...]

   81      function bridge(
   82          uint256 amt2Bridge,
   83          SwapInstructions memory postBridge,
   84          uint256 dstChainId,
   85          address target,
   86          address paymentOperator,
   87          bytes memory payload,
   88          bytes calldata additionalArgs,
   89          address payable refund
   90      ) public payable onlyUtb returns (bytes memory bridgePayload) {
   91          require(
   92              destinationBridgeAdapter[dstChainId] != address(0),
   93              string.concat("dst chain address not set ")
   94          );
   95  
   96          uint64 dstGas = abi.decode(additionalArgs, (uint64));
   97  
   98          bridgePayload = abi.encodeCall(
   99              this.receiveFromBridge,
   100              (postBridge, target, paymentOperator, payload, refund)
   101          );
   102  
   103          SwapParams memory swapParams = abi.decode(
   104              postBridge.swapPayload,
   105              (SwapParams)
   106          );
   107  
-  108          if (!gasIsEth) {
+               if (gasIsNotEth) {
   109              IERC20(bridgeToken).transferFrom(
   110                  msg.sender,
   111                  address(this),
   112                  amt2Bridge
   113              );
   114              IERC20(bridgeToken).approve(address(router), amt2Bridge);
   115          }
   116  
   117          router.bridgeWithPayload{value: msg.value}(
   118              lzIdLookup[dstChainId],
   119              destinationBridgeAdapter[dstChainId],
   120              swapParams.amountIn,
   121              false,
   122              dstGas,
   123              bridgePayload
   124          );
   125      }

```

*Code link:* [[18-125](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L18-L125)]



***
&nbsp;

                
### [GO-09] Avoid defining useless parameters in `getBridgedAmount()` function in StargateBridgeAdapter.sol
                


*Issue's instances:* 1

#### File: [StargateBridgeAdapter.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol)

*Instances:* 1    


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

*Code link:* [[61-67](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L61-L67)]



***
&nbsp;

                
### [GO-10] Use alternative formulation in order to avoid NOT using AND instead of OR or vice versa
                
It's possible to change boolean formula in order to use AND instead of OR or vice versa.

In this way, ```>``` could be substituted with ```>=```, ```!=``` with ```==```

 In general

#### Two variables

```
(a || b) = !(!(a || b)) = !(!a && !b)
```

| a   | b   | (a or b)  | !a and !b  | !(!a and !b)|
|:---:|:---:|:---------:|:----------:|:------------:|
| 0  | 0  | 0         | 1          | 0            |
| 0  | 1  | 1         | 0          | 1            |
| 1  | 0  | 1         | 0          | 1            |
| 1  | 1  | 1         | 0          | 1            |


#### Three variables

```
(a || b || c) = !(!(a || b || c)) = !(!a && !b && !c)
```

| a   | b   | c   | (a or b or c)  | !a and !b and !c  | !(!a and !b and !c)|
|:---:|:---:|:---:|:--------------:|:-----------------:|:------------------:|
| 0  | 0  | 0 | 0              | 1                 | 0                  |
| 0  | 0  | 1 | 1              | 0                 | 1                  |
| 0  | 1  | 0 | 1              | 0                 | 1                  |
| 0  | 1  | 1 | 1              | 0                 | 1                  |
| 1  | 0  | 0 | 1              | 0                 | 1                  |
| 1  | 0  | 1 | 1              | 0                 | 1                  |
| 1  | 1  | 0 | 1              | 0                 | 1                  |
| 1  | 1  | 1 | 1              | 0                 | 1                  |

*Issue's instances:* 2

#### File: [DecentBridgeExecutor.sol](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol)

*Instances:* 1    


```diff
File: DecentBridgeExecutor.sol

Current Code:
-  77          if (!gasCurrencyIsEth || !deliverEth) {
-  78              _executeWeth(from, target, amount, callPayload);
-  79          } else {
-  80              _executeEth(from, target, amount, callPayload);
-  81          }

New Code:
+  77          if (gasCurrencyIsEth && deliverEth) {
+  78              _executeEth(from, target, amount, callPayload);
+  79          } else {
+  80              _executeWeth(from, target, amount, callPayload);
+  81          }

```

*Code link:* [[77-81](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L77-L81)]



***
&nbsp;

#### File: [DecentEthRouter.sol](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol)

*Instances:* 1    


```diff
File: DecentEthRouter.sol

Current Code:
-  272              if (!gasCurrencyIsEth || !deliverEth) {
-  273                  weth.transfer(_to, _amount);
-  274              } else {
-  275                  weth.withdraw(_amount);
-  276                  payable(_to).transfer(_amount);
-  277              }

New Code:
+  272              if (gasCurrencyIsEth && deliverEth) {
+  273                  weth.withdraw(_amount);
+  274                  payable(_to).transfer(_amount);
+  275              } else {
+  276                  weth.transfer(_to, _amount);
+  277              }

```

*Code link:* [[272-277](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L272-L277)]



***
