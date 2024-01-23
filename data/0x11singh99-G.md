## Gas Optimizations

**Note:  It includes only those instances which were Missed by bot-report.**

| Issue | Gas Saved |
|-|:-:|
| [[G-01] **`State` variables only set in the `constructor` should be declared `immutable`(Missed by bot)**](#g-01-state-variables-only-set-in-the-constructor-should-be-declared-immutablemissed-by-bot) | ~8000 |
| [[G-02] **Set `wrapped` variable address in `constructor` instead of function and make it `immutable`**](#g-02-set-wrapped-variable-address-in-constructor-instead-of-function-and-make-it-immutable) | ~124000 |
| [[G-03] **Remove unnecessary stack variable creation**](#g-03-remove-unnecessary-stack-variable-creation) | 10 |
| [[G-04] **Remove unnecessary `modifier`**](#g-04-remove-unnecessary-modifier) | ~60000 |
| [[G-05] **Function should be mark `payable` to save gas**](#g-05-function-should-be-mark-payable-to-save-gas) | 144 |
| [[G-06] **Don't cache value if it is used only once**](#g-06-dont-cache-value-if-it-is-used-only-once) | 20 |
| [[G-07] **Use `calldata` instead of `memory`**](#g-07-use-calldata-instead-of-memory) | ~2380 |
| [[G-08] **Remove unnecessary external pure functions**](#g-08-remove-unnecessary-external-pure-functions) | - |
| [[G-09] **Remove unused import**](#g-09-remove-unused-import) | - |
| [[G-10] **Check amount for zero value**](#g-10-check-amount-for-zero-value) | - |
| [[G-11] **If any operation can't be `underflow`/`overflow` should be used unchecked**](#g-11-if-any-operation-cant-be-underflowoverflow-should-be-used-unchecked) | - |
| [[G-12] **Use already cached value**](#g-12-use-already-cached-value) | - |
| [[G-13] **Cache state variable rather than re-reading from storage**](#g-13-cache-state-variable-rather-than-re-reading-from-storage) | - |
| [[G-14] **Cache `address(this)` instead of re-calculate**](#g-14-cache-addressthis-instead-of-re-calculate) | - |
| [[G-15] **Refactor `if-else` statement to save gas**](#g-15-refactor-if-else-statement-to-save-gas) | - |
| [[G-16] **Refactor function `onOFTReceived` to save gas**](#g-16-refactor-function-onoftreceived-to-save-gas) | - |

## [G-01] `State` variables only set in the `constructor` should be declared `immutable`(Missed by bot)

**Note: It includes only those instances which were Missed by bot-report**


Accessing state variables within a function involves an SLOAD operation, but `immutable` variables can be accessed directly without the need of it, thus reducing gas costs. As these state variables are assigned only in the constructor, consider declaring them `immutable`.
Avoids a Gsset(** 20000 gas**) in the constructor, and replaces the first access in each transaction(Gcoldsload - ** 2100 gas **) and each access thereafter(Gwarmacces - ** 100 gas ) with aPUSH32( 3 gas **).

_4 instances_
### Total Gas Saved ~8000


```solidity
File : src/DecentBridgeExecutor.sol

09:     IWETH weth;
10:     bool public gasCurrencyIsEth;

```
[09-10](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L9C4-L10C34)

```solidity
File : src/bridge_adapters/DecentBridgeAdapter.sol

19:     address bridgeToken;

```
[19](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L19C4-L19C25)

```solidity
File : src/DecentEthRouter.sol

15:    IDecentBridgeExecutor public executor;

```
[15](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L15C5-L15C43)

## [G-02] Set `wrapped` variable address in `constructor` instead of function and make it `immutable`

`wrapped` is referring to ERC20 version of native currency of that chain. like WETH for Ethereum Mainnet.
So they are one per chain and fixed contracts on every chain. So they can be marked immutable since these contracts using them will be deployed on different chains separately.

Avoids a Gsset(**20000 gas**) in the constructor, and replaces the first access in each transaction(Gcoldsload - **2100 gas**) and each access thereafter(Gwarmacces - **100 gas** ) with aPUSH32( **3 gas**)

_2 instances_
### Total Gas Saved ~120,000 Deployments Gas and Gsset(**20000 gas**) in the constructor Gcoldsload - **2100 gas**.

### Set `address wrapped` in the constructor and make it immutable and remove the setter function `setWrapped`

```solidity
File : src/swappers/UniSwapper.sol

    constructor() UTBOwned() {}
...
    address payable public wrapped;

...
    function setWrapped(address payable _wrapped) public onlyOwner {
        wrapped = _wrapped;
    }

```
[14-26](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L14C1-L26C6)

```diff
File : src/swappers/UniSwapper.sol

-   constructor() UTBOwned() {}

    uint8 public constant SWAPPER_ID = 0;
    address public uniswap_router;
-   address payable public wrapped;

+   address payable public immutable wrapped;

+   constructor(address payable _wrapped) UTBOwned() {
+       wrapped = _wrapped; 
+ }

    function setRouter(address _router) public onlyOwner {
        uniswap_router = _router;
    }

-    function setWrapped(address payable _wrapped) public onlyOwner {
-        wrapped = _wrapped;
-    }

```

### Set `wrapped` in the constructor and make it immutable and remove the setter function `setWrapped`

```solidity
File : src/UTB.sol

     constructor() Owned(msg.sender) {}
...
     IWETH wrapped;
  
...
     function setWrapped(address payable _wrapped) public onlyOwner {
        wrapped = IWETH(_wrapped);
    }

```
[16-38](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L16C5-L38C6)

```diff
File : src/UTB.sol

-   constructor() Owned(msg.sender) {}

    IUTBExecutor executor;
    IUTBFeeCollector feeCollector;
-   IWETH wrapped;
+   IWETH immutable wrapped;
    mapping(uint8 => address) public swappers;
    mapping(uint8 => address) public bridgeAdapters;

+   constructor(address payable _wrapped) Owned(msg.sender) {
+        wrapped = IWETH(_wrapped);
+   }


    /**
     * @dev Sets the executor.
     * @param _executor The address of the executor.
     */
    function setExecutor(address _executor) public onlyOwner {
        executor = IUTBExecutor(_executor);
    }

    /**
     * @dev Sets the wrapped native token.
     * @param _wrapped The address of the wrapped token.
     */
-    function setWrapped(address payable _wrapped) public onlyOwner {
-        wrapped = IWETH(_wrapped);
-    }

```

## [G-03] Remove unnecessary stack variable creation 

If you want to avoid magic numbers then use constant

_1 instance_
### Total Gas Saved ~10

```solidity
File : src/DecentEthRouter.sol

96:        uint256 GAS_FOR_RELAY = 100000;
97:        uint256 gasAmount = GAS_FOR_RELAY + _dstGasForCall;

```
[96-97](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L96C1-L97C60)

## [G-04] Remove unnecessary `modifier`

_2 instances_
### Total Gas Saved ~60000

### Unnecessary `onlyIfWeHaveEnoughReserves(amount)` modifier check 

 `onlyIfWeHaveEnoughReserves` modifier is checking that our `DecentEthRouter` contract have enough weth or not. Since It will be checked in Weth contract also when this `DecentEthRouter` will call withdraw or transfer function in weth contract. And when this caller contract don't have enough reserves `WETH` contract will revert automatically which will revert entire transaction. So there is no need to add extra modifier check here in our contract checking same thing. 
 It will save gas for not using that modifier and deleting this modifier will also some save deployment gas also.

### Saves ~30000 GAS

It will save the gas every time `redeemEth` function called.

```solidity
File : src/DecentEthRouter.sol

285:    function redeemEth(
286:        uint256 amount
287:    ) public onlyIfWeHaveEnoughReserves(amount) {
288:        dcntEth.transferFrom(msg.sender, address(this), amount);
289:        weth.withdraw(amount);
290:        payable(msg.sender).transfer(amount);
291:    }

```
[285-291](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L285C5-L291C6)

```diff
File : src/DecentEthRouter.sol

        function redeemEth(
        uint256 amount
-     ) public onlyIfWeHaveEnoughReserves(amount) {
+     ) public {
+       weth.withdraw(amount);
        dcntEth.transferFrom(msg.sender, address(this), amount);
-       weth.withdraw(amount);
        payable(msg.sender).transfer(amount);
    }

```

### Saves ~30000 GAS

It will save the gas every time `redeemWeth` function called.

```solidity
File : src/DecentEthRouter.sol

294:    function redeemWeth(
295:        uint256 amount
296:    ) public onlyIfWeHaveEnoughReserves(amount) {
297:        dcntEth.transferFrom(msg.sender, address(this), amount);
298:        weth.transfer(msg.sender, amount);
299:    }

```
[294-299](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L294C1-L299C6)

```diff
File : src/DecentEthRouter.sol

    function redeemWeth(
     uint256 amount
-    ) public onlyIfWeHaveEnoughReserves(amount) {
+    ) public {
+       weth.transfer(msg.sender, amount);
        dcntEth.transferFrom(msg.sender, address(this), amount);
-       weth.transfer(msg.sender, amount);
    }

```

## [G-05] Function should be mark `payable` to save gas

**Note: It includes only those instances which were Missed by bot-report**

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

_6 instances_
### Total Gas Saved ~24*6 => ~144 GAS
```solidity
File : src/DcntEth.sol

24:    function mint(address _to, uint256 _amount) public onlyRouter {    

28:    function burn(address _from, uint256 _amount) public onlyRouter {

32:    function mintByOwner(address _to, uint256 _amount) public onlyOwner {

36:    function burnByOwner(address _from, uint256 _amount) public onlyOwner {

```
[24](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L24C5-L24C68), [28](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L28C5-L28C70), [32](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L32C5-L32C74), [36](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L36C5-L36C76)

```solidity
File : src/UTBFeeCollector.sol

18:   function setSigner(address _signer) public onlyOwner {

69:   function redeemFees(address token, uint amount) public onlyOwner {   

```
[18](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L18C5-L18C59), [69](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L69C5-L69C71)

## [G-06] Don't cache value if it is used only once

If a value is only intended to be used once then it should not be cached. Caching the value will result in unnecessary stack manipulation.

_2 instances_
### Total Gas Saved  ~20 Gas

### Don't cache `ISwapper(swapper)` and `IBridgeAdapter(bridge)`

```solidity
File : src/UTB.sol

326:     ISwapper s = ISwapper(swapper);
327:       swappers[s.getId()] = swapper;

335:     IBridgeAdapter b = IBridgeAdapter(bridge);
336:        bridgeAdapters[b.getId()] = bridge;

```
[326-327](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L326C8-L327C39), [335-336](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L335C9-L336C44)

```diff
File : src/UTB.sol

-       ISwapper s = ISwapper(swapper);
-        swappers[s.getId()] = swapper;
+        swappers[ISwapper(swapper).getId()] = swapper;

-      IBridgeAdapter b = IBridgeAdapter(bridge);
-        bridgeAdapters[b.getId()] = bridge;
+        bridgeAdapters[IBridgeAdapter(bridge).getId()] = bridge;

```

## [G-07] Use `calldata` instead of `memory`

**Note: Missed by bot-report**

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

_5 instances_
### Total Gas Saved  ~ 476*5 = ~ 2380 Gas

```solidity
File : src/DecentEthRouter.sol

113:    function estimateSendAndCallFee(
114:         uint8 msgType,
115:         uint16 _dstChainId,
116:         address _toAddress,
117:         uint _amount,
118:         uint64 _dstGasForCall,
119:         bool deliverEth,
120:         bytes memory payload //@audit use calldata
121:     ) public view returns (uint nativeFee, uint zroFee) {


197:    function bridgeWithPayload(
198:         uint16 _dstChainId,
199:         address _toAddress,
200:         uint _amount,
201:         bool deliverEth,
202:         uint64 _dstGasForCall,
203:         bytes memory additionalPayload //@audit use calldata
204:     ) public payable {


237:     function onOFTReceived(
238:         uint16 _srcChainId,
239:         bytes calldata,
240:         uint64,
241:         bytes32,
242:         uint _amount,
243:         bytes memory _payload //@audit use calldata
244:     ) external override onlyLzApp {  

```
[113-121](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L113C5-L121C58), [197-204](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L197C1-L204C23), [237-244](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L237C1-L244C36)

```solidity
File : src/UTBFeeCollector.sol

44:     function collectFees(
45:          FeeStructure calldata fees,
46:          bytes memory packedInfo, //@audit use calldata
47:          bytes memory signature //@audit use calldata
48:      ) public payable onlyUtb {

```
[44-48](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L44C4-L48C31)

## [G-08] Remove unnecessary external pure functions

Since these are external pure functions so neither they can be used in this contract nor they telling something about contract. So it is unnecessary to have external pure functions in contract purely for calculation purpose. If it is called by another contract than it will be better they do it in their contract. If it is called from outside  then it will be better to do it there. Since they are not serving any purpose related to smart contract so they can be removed.

_2 instances_


```solidity
File : src/bridge_adapters/StargateBridgeAdapter.sol

55:    function getBridgeToken(

61:    function getBridgedAmount(        

```
[55](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L55C5-L55C29), [61](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L61C5-L61C31)

## [G-09] Remove unused import

**Note: It includes only those instances which were Missed by bot-report**

_2 instances_

```solidity
File : src/UTBFeeCollector.sol

      //@audit `SwapInstructions` and `BridgeInstructions`
05:   import {SwapInstructions, FeeStructure, BridgeInstructions} from "./CommonTypes.sol";

```
[05](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L5C1-L5C86)


## [G-10] Check amount for zero value

**Note: It includes only those instances which were Missed by bot-report**

_6 instances_

```solidity
File : src/DcntEth.sol

24:    function mint(address _to, uint256 _amount) public onlyRouter {
25:          _mint(_to, _amount); //@audit check `_amount` for zero
26:     }
27:
28:    function burn(address _from, uint256 _amount) public onlyRouter {
29:         _burn(_from, _amount); //@audit check `_amount` for zero
30:     }
31:
32:    function mintByOwner(address _to, uint256 _amount) public onlyOwner {
33:         _mint(_to, _amount); //@audit check `_amount` for zero
34:    }
35:
36:    function burnByOwner(address _from, uint256 _amount) public onlyOwner {
37:         _burn(_from, _amount); //@audit check `_amount` for zero
38:    }

```
[24-38](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L24C5-L38C6)

```solidity
File : src/UTBFeeCollector.sol

70:     if (token == address(0)) {
71:            payable(owner).transfer(amount); //@audit check `amount` for zero
72:     } else {
73:            IERC20(token).transfer(owner, amount); //@audit check `amount` for zero
74:        }

```
[70-74](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L70C9-L74C10)

## [G-11] If any operation can't be `underflow`/`overflow` should be used unchecked

**Note: It includes only those instances which were Missed by bot-report**

_2 instances_

```solidity
File : src/DecentEthRouter.sol

61:        uint256 balance = balanceOf[msg.sender];
62:        require(balance >= amount, "not enough balance");
63:        _;
64:        balanceOf[msg.sender] -= amount;
65:    }

```
[61-65](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L61C1-L65C6)

```solidity
File : src/DecentEthRouter.sol

          //@audit can be marked unchecked since first is `100000` and second is uint64 so no chance to overflow
97:       uint256 gasAmount = GAS_FOR_RELAY + _dstGasForCall;

```
[97](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L97C2-L97C60)

## [G-12] Use already cached value

_2 instances_

```solidity
File : src/DecentEthRouter.sol

61:        uint256 balance = balanceOf[msg.sender];
62:        require(balance >= amount, "not enough balance");
63:        _;
64:        balanceOf[msg.sender] -= amount;
65:    }

```
[61-65](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L61C1-L65C6)

```diff
File : src/DecentEthRouter.sol

         uint256 balance = balanceOf[msg.sender];
         require(balance >= amount, "not enough balance");
         _;
-        balanceOf[msg.sender] -= amount;
+        balanceOf[msg.sender] = balance - amount;
    }

```



## [G-13] Cache state variable rather than re-reading from storage

**Note: It includes only those instances which were Missed by bot-report**

_6 instances_

### Cache `weth`

```solidity
File : src/DecentBridgeExecutor.sol

30:        uint256 balanceBefore = weth.balanceOf(address(this));
31:        weth.approve(target, amount);
32:
33:        (bool success, ) = target.call(callPayload);
34:
35:        if (!success) {
36:            weth.transfer(from, amount);
37:            return;
38:        }
39:
40:        uint256 remainingAfterCall = amount -
41:            (balanceBefore - weth.balanceOf(address(this)));
42:
43:        // refund the sender with excess WETH
44:        weth.transfer(from, remainingAfterCall);

```
[30-44](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L30C9-L44C49)

```diff
File : src/DecentBridgeExecutor.sol

+       IWETH _weth = weth;
-       uint256 balanceBefore = weth.balanceOf(address(this));
-       weth.approve(target, amount);
+       uint256 balanceBefore = _weth.balanceOf(address(this));
+       _weth.approve(target, amount);

        (bool success, ) = target.call(callPayload);

        if (!success) {
-            weth.transfer(from, amount);
+            _weth.transfer(from, amount);
             return;
        }

        uint256 remainingAfterCall = amount -
-            (balanceBefore - weth.balanceOf(address(this)));
+            (balanceBefore - _weth.balanceOf(address(this)));

        // refund the sender with excess WETH
-        weth.transfer(from, remainingAfterCall);
+        _weth.transfer(from, remainingAfterCall);

```

### Cache `weth`

```solidity
File : src/DecentEthRouter.sol

266:        if (weth.balanceOf(address(this)) < _amount) {
267:            dcntEth.transfer(_to, _amount);
268:            return;
269:        }
270:
271:        if (msgType == MT_ETH_TRANSFER) {
272:            if (!gasCurrencyIsEth || !deliverEth) {
273:                weth.transfer(_to, _amount);
274:            } else {
275:                weth.withdraw(_amount);
276:                payable(_to).transfer(_amount);
277:            }
278:        } else {
289:            weth.approve(address(executor), _amount);

```
[266-279](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L266C9-L279C54)

```diff
File : src/DecentEthRouter.sol

+       IWETH _weth = weth;
-       if (weth.balanceOf(address(this)) < _amount) {
+       if (_weth.balanceOf(address(this)) < _amount) {
            dcntEth.transfer(_to, _amount);
            return;
        }

        if (msgType == MT_ETH_TRANSFER) {
            if (!gasCurrencyIsEth || !deliverEth) {
-               weth.transfer(_to, _amount);
+               _weth.transfer(_to, _amount);
            } else {
-               weth.withdraw(_amount);
+               _weth.withdraw(_amount);
                payable(_to).transfer(_amount);
            }
        } else {
-            weth.approve(address(executor), _amount);
+            _weth.approve(address(executor), _amount);

```

### Cache `executor`

```solidity
File : src/DecentEthRouter.sol

279:    weth.approve(address(executor), _amount);
280:    executor.execute(_from, _to, deliverEth, _amount, callPayload);

```
[279-280](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L279C12-L280C76)

```diff
File : src/DecentEthRouter.sol

+       IDecentBridgeExecutor _executor = executor;
-       weth.approve(address(executor), _amount);
-       executor.execute(_from, _to, deliverEth, _amount, callPayload);
+       weth.approve(address(_executor), _amount);
+       _executor.execute(_from, _to, deliverEth, _amount, callPayload);

```

### Cache `wrapped`

```solidity
File : src/UTB.sol

74:     if (swapParams.tokenIn == address(0)) {
75:            require(msg.value >= swapParams.amountIn, "not enough native");
76:            wrapped.deposit{value: swapParams.amountIn}();
77:            swapParams.tokenIn = address(wrapped);
78:            swapInstructions.swapPayload = swapper.updateSwapParams(
79:                swapParams,
80:                swapInstructions.swapPayload
81:            );
82:        } else if (retrieveTokenIn) {
83:            IERC20(swapParams.tokenIn).transferFrom(
84:                msg.sender,
85:                address(this),
86:                swapParams.amountIn
87:            );
88:        }
89:
90:        IERC20(swapParams.tokenIn).approve(
91:            address(swapper),
92:            swapParams.amountIn
93:        );
94:
95:        (tokenOut, amountOut) = swapper.swap(swapInstructions.swapPayload);
96:
97:        if (tokenOut == address(0)) {
98:            wrapped.withdraw(amountOut);

```
[74-98](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L74C9-L98C41)

```diff
File : src/UTB.sol

+       IWETH _wrapped = wrapped;
        if (swapParams.tokenIn == address(0)) {
            require(msg.value >= swapParams.amountIn, "not enough native");
-           wrapped.deposit{value: swapParams.amountIn}();
-           swapParams.tokenIn = address(wrapped);
+           _wrapped.deposit{value: swapParams.amountIn}();
+           swapParams.tokenIn = address(_wrapped);
            swapInstructions.swapPayload = swapper.updateSwapParams(
                swapParams,
                swapInstructions.swapPayload
            );
        } else if (retrieveTokenIn) {
            IERC20(swapParams.tokenIn).transferFrom(
                msg.sender,
                address(this),
                swapParams.amountIn
            );
        }

        IERC20(swapParams.tokenIn).approve(
            address(swapper),
            swapParams.amountIn
        );

        (tokenOut, amountOut) = swapper.swap(swapInstructions.swapPayload);

        if (tokenOut == address(0)) {
-            wrapped.withdraw(amountOut);
+            _wrapped.withdraw(amountOut);

```

### Cache `executor`

```solidity
File : src/UTB.sol

152:    IERC20(tokenOut).approve(address(executor), amountOut);
153:            executor.execute(

```
[152-153](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L152C13-L153C30)

```diff
File : src/UTB.sol

+       IUTBExecutor _executor = executor;
-        IERC20(tokenOut).approve(address(executor), amountOut);
-            executor.execute(
+        IERC20(tokenOut).approve(address(_executor), amountOut);
+            _executor.execute(

```

### Cache `feeCollector`

```solidity
File : src/UTB.sol

233:    if (address(feeCollector) != address(0)) {
234:            uint value = 0;
235:            if (fees.feeToken != address(0)) {
236:                IERC20(fees.feeToken).transferFrom(
237:                    msg.sender,
238:                    address(this),
239:                    fees.feeAmount
240:                );
241:                IERC20(fees.feeToken).approve(
242:                    address(feeCollector),
243:                    fees.feeAmount
244:                );
245:            } else {
246:                value = fees.feeAmount;
247:            }
248:            feeCollector.collectFees{value: value}(fees, packedInfo, signature);

```
[233-248](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L233C9-L248C81)

```diff
File : src/UTB.sol

+       IUTBFeeCollector _feeCollector = feeCollector;
-        if (address(feeCollector) != address(0)) {
+        if (address(_feeCollector) != address(0)) {
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

```

## [G-14] Cache `address(this)` instead of re-calculate

Calculating `address(this)` takes some gas so it is better to cache it instead of re-calculating multiple times in same function.
_2 instances_

```solidity
File : src/DecentBridgeExecutor.sol

30:        uint256 balanceBefore = weth.balanceOf(address(this));
31:        weth.approve(target, amount);
32:
33:        (bool success, ) = target.call(callPayload);
34:
35:        if (!success) {
36:            weth.transfer(from, amount);
37:            return;
38:        }
39:
40:        uint256 remainingAfterCall = amount -
41:            (balanceBefore - weth.balanceOf(address(this)));

```
[30-41](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L30C9-L41C61)

```diff
File : src/DecentBridgeExecutor.sol

+       address _this = address(this);
-       uint256 balanceBefore = weth.balanceOf(address(this));
+       uint256 balanceBefore = weth.balanceOf(_this);
        weth.approve(target, amount);

        (bool success, ) = target.call(callPayload);

        if (!success) {
            weth.transfer(from, amount);
            return;
        }

        uint256 remainingAfterCall = amount -
-            (balanceBefore - weth.balanceOf(address(this)));
+            (balanceBefore - weth.balanceOf(_this));

```

```solidity
File : src/UTBExecutor.sol

59:        uint initBalance = IERC20(token).balanceOf(address(this));
60:
61:        IERC20(token).transferFrom(msg.sender, address(this), amount);
62:        IERC20(token).approve(paymentOperator, amount);
63:
64:        if (extraNative > 0) {
65:            (success, ) = target.call{value: extraNative}(payload);
66:            if (!success) {
67:                (refund.call{value: extraNative}(""));
68:            }
69:        } else {
70:            (success, ) = target.call(payload);
71:        }
72:
73:        uint remainingBalance = IERC20(token).balanceOf(address(this)) -
74:            initBalance;

```
[59-74](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L59C9-L74C25)

```diff
File : src/UTBExecutor.sol

+       address _this = address(this);
-       uint initBalance = IERC20(token).balanceOf(address(this));
+       uint initBalance = IERC20(token).balanceOf(_this);

-       IERC20(token).transferFrom(msg.sender, address(this), amount);
+       IERC20(token).transferFrom(msg.sender, _this, amount);
        IERC20(token).approve(paymentOperator, amount);

        if (extraNative > 0) {
            (success, ) = target.call{value: extraNative}(payload);
            if (!success) {
                (refund.call{value: extraNative}(""));
            }
        } else {
            (success, ) = target.call(payload);
        }

-        uint remainingBalance = IERC20(token).balanceOf(address(this)) -
+        uint remainingBalance = IERC20(token).balanceOf(_this) -
            initBalance;

```

## [G-15] Refactor `if-else` statement to save gas

Since it is better to write less gas consuming condition before more gas consumer. In if statement due to operator short-circuiting. First is evaluated if 2nd is not needed then 2nd will not be evaluated in || OR && operations.

To save 2 not operations and 1 Sload  if deliveryEth is false we can refactor this below code.

_2 instances_

```solidity
File : src/DecentBridgeExecutor.sol

77:     if (!gasCurrencyIsEth || !deliverEth) {
78:           _executeWeth(from, target, amount, callPayload);
79:      } else {
80:           _executeEth(from, target, amount, callPayload);
81:      }

```
[77-81](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L77C8-L81C10)

```diff
File : src/DecentBridgeExecutor.sol

-       if (!gasCurrencyIsEth || !deliverEth) {
-            _executeWeth(from, target, amount, callPayload);
-        } else {
-            _executeEth(from, target, amount, callPayload);
-        }
+       if (deliverEth && gasCurrencyIsEth) {
+            _executeEth(from, target, amount, callPayload);
+       } else {
+            _executeWeth(from, target, amount, callPayload);
+       }

```

```solidity
File : src/DecentEthRouter.sol

272:        if (!gasCurrencyIsEth || !deliverEth) {
273:                weth.transfer(_to, _amount);
274:        } else {
275:                weth.withdraw(_amount);
276:                payable(_to).transfer(_amount);
277:        }

```
[272-277](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L272C13-L277C14)

```diff
File : src/DecentEthRouter.sol

-        if (!gasCurrencyIsEth || !deliverEth) {
-             weth.transfer(_to, _amount);
-        } else {
-            weth.withdraw(_amount);
-            payable(_to).transfer(_amount);
-        }
+        if (deliverEth && gasCurrencyIsEth) {
+            weth.withdraw(_amount);
+            payable(_to).transfer(_amount); 
+        } else {
+             weth.transfer(_to, _amount);
+         }      

```

## [G-16] Refactor function `onOFTReceived` to save gas

We can refactor this function to avoid 1 abi.decode. It will have same impact.
_1 instance_

```solidity
File : src/DecentEthRouter.sol

245:    (uint8 msgType, address _from, address _to, bool deliverEth) = abi
246:            .decode(_payload, (uint8, address, address, bool));
247:
248:        bytes memory callPayload = "";
249:
250:        if (msgType == MT_ETH_TRANSFER_WITH_PAYLOAD) {
251:            (, , , , callPayload) = abi.decode(
252:                _payload,
253:                (uint8, address, address, bool, bytes)
254:            );

```
[245-254](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L245C8-L254C15)

```diff
File : src/DecentEthRouter.sol

-        (uint8 msgType, address _from, address _to, bool deliverEth) = abi
-            .decode(_payload, (uint8, address, address, bool));

         bytes memory callPayload = "";

         if (msgType == MT_ETH_TRANSFER_WITH_PAYLOAD) {
-            (, , , , callPayload) = abi.decode(
-                _payload,
-                (uint8, address, address, bool, bytes)
-            );
+            (uint8 msgType, address _from, address _to, bool deliverEth, callPayload) = abi.decode(
+                _payload,
+                (uint8, address, address, bool, bytes)
+            );
+        }
+        else{
+             (uint8 msgType, address _from, address _to, bool deliverEth) = abi
+            .decode(_payload, (uint8, address, address, bool));
+        }

```


