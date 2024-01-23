## [G-01] Empty blocks should be removed or emit 

```
   constructor() Owned(msg.sender) {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L16C2-L16C39

```
    receive() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L339C1-L339C34

```
 fallback() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L341C4-L341C35

```
 constructor() Owned(msg.sender) {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L8C4-L8C39

```
  constructor() UTBOwned() {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L12C3-L12C32

```
 receive() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L77C4-L77C34

```
fallback() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L79C5-L79C35

```
   constructor() UTBOwned() {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L9C2-L9C32

```
 receive() external payable {}

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L156C4-L157C1

```
fallback() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L158C5-L158C35

```
 constructor() BaseAdapter() {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L29C4-L29C35

```
 receive() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L218C4-L218C34

```
fallback() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L220C5-L220C35

```
   constructor() UTBOwned() {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L14C2-L14C32

```
 receive() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L171

```
 fallback() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L173C4-L173C35

```
   receive() external payable {}
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L337C2-L337C34

```
 fallback() external payable {}
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L339C4-L339C35

```
 receive() external payable {}
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L85C4-L85C34

```
 fallback() external payable {}
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L88C4-L88C35

## [G02] address(this) should be cached

```
  IERC20(token).transferFrom(msg.sender, address(this), amount);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L61C7-L61C71

```
   IERC20(fees.feeToken).transferFrom(
                utb,
                address(this),
                fees.feeAmount
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L56C10-L59C31

```
IERC20(swapParams.tokenIn).transferFrom(
            msg.sender,
            address(this),
            swapParams.amountIn
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L139C9-L142C32

```
  IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88C7-L88C81

```
 require(weth.balanceOf(address(this)) > amount, "not enough reserves");
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L51C8-L51C80

```
  weth.transferFrom(msg.sender, address(this), _amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L181C11-L181C67

```
       address(this), // from address that has dcntEth (so DecentRouter)
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L186C6-L186C78

```
 if (weth.balanceOf(address(this)) < _amount) {
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L266C8-L266C55

```
 dcntEth.transferFrom(msg.sender, address(this), amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L288C8-L288C65

```
 dcntEth.transferFrom(msg.sender, address(this), amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L297C8-L297C65

```
  dcntEth.mint(address(this), msg.value);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L309C7-L309C48

```
  dcntEth.burn(address(this), amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L316C7-L316C45

```
 weth.transferFrom(msg.sender, address(this), amount);
        dcntEth.mint(address(this), amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L325C8-L326C45

```
   dcntEth.burn(address(this), amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L333C6-L333C45

```
  weth.transferFrom(msg.sender, address(this), amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L75C7-L75C62

## [G-03] Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

We can combine multiple mappings below into structs. This will result in cheaper storage reads since
multiple mappings are accessed in functions and those values are now occupying the same storage
slot, meaning the slot will become warm after the first SLOAD. In addition, when writing to and reading
from the struct values we will avoid a Gsset (20000 gas) and Gcoldsload (2100 gas) since multiple
struct values are now occupying the same slot.

```
   mapping(uint256 => address) public destinationBridgeAdapter;
    IDecentEthRouter public router;
    mapping(uint256 => uint16) lzIdLookup;
    mapping(uint16 => uint256) chainIdLookup;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L14C2-L17C46


## [G-04] x += y/x -= y costs more gas than x = x + y/x = x - y for state variables

```
  balanceOf[msg.sender] += amount;
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L56C7-L56C41

```
 balanceOf[msg.sender] -= amount;
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L64C8-L64C41

## [G-05] require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a SLOAD (2100 gas for the 1st one) in a function that may ultimately revert in the unhappy case.

```
   modifier onlyEthChain() {
        require(gasCurrencyIsEth, "Gas currency is not ETH");
        _;
    }

    modifier onlyLzApp() {
        require(
            address(dcntEth) == msg.sender,
            "DecentEthRouter: only lz App can call"
        );
        _;
    }
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L37C2-L48C6

## [G-06] Unchecking arithmetics operations that can’t underflow/overflow

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block: https://docs.soliditylang.org/en/v0.8.10/control-structures.html#checked-or-unchecked-arithmetic

As the following only concerns substractions with owner supplied value, it’s not a stretch to consider that they are safe/trusted enough be wrapped with an unchecked block (around 25 gas saved per instance):

```
  uint remainingBalance = IERC20(token).balanceOf(address(this)) -
            initBalance;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L73C7-L74C25

```
  if (gasCurrencyIsEth) {
            weth.deposit{value: _amount}();
            gasValue = msg.value - _amount;
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L177C7-L179C44

```
  uint256 remainingAfterCall = amount -
            (balanceBefore - weth.balanceOf(address(this)));

```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L40C7-L42C1

## [G-07] Upgrade pragma

Using newer compiler versions and the optimizer give gas optimizations. Also, additional safety checks are available for free.

The advantages here are:

Low level inliner (>= 0.8.2): Cheaper runtime gas (especially relevant when the contract has small functions).
Optimizer improvements in packed structs (>= 0.8.3)
Custom errors (>= 0.8.4): cheaper deployment cost and runtime cost. Note: the runtime cost is only relevant when the revert condition is met. In short, replace revert strings by custom errors.
Contract existence checks (>= 0.8.10): external calls skip contract existence checks if the external call has a return value
Consider upgrading here :

```
pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L2

```
pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L2C1-L2C24

```
pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L2C1-L2C24

```
pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L2C1-L2C24

```
pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L2C1-L2C24

```
pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L2C1-L2C24

```
pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L2C1-L2C24

```
pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L2C1-L2C24

```
pragma solidity ^0.8.0;
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L40C7-L42C1

## [GAS-08] Use shift Right/Left instead of division/multiplication if possible

```
 ) external pure returns (uint256) {
        return (amt2Bridge * (1e4 - SG_FEE_BPS)) / 1e4;
    }
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L65C4-L67C6

```
   (amt2Bridge * (10000 - SG_FEE_BPS)) / 10000, // the min qty you would accept on the destination, fee is 6 bips
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L176C10-L176C123

## [GAS-09] Don't initialize variables with default value

```
 uint value = 0;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L234C12-L234C28

```
  uint8 public constant BRIDGE_ID = 0;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L13C3-L13C41

```
  uint8 constant EXACT_IN = 0;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L5C3-L5C33

```
 uint8 public constant SWAPPER_ID = 0;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L16C4-L16C42

```
uint8 public constant MT_ETH_TRANSFER = 0;
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L17C5-L17C47


## [GAS-10] Long revert strings

```
   "Only bridge executor can call this"
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L14C10-L14C49

```
require(
            address(dcntEth) == msg.sender,
            "DecentEthRouter: only lz App can call"
        );
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L43C9-L46C11

## [G-11] `variable == false` instead of `!variable`.
a bit cheapier when you replace:

```
   if (!success) {
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L53C10-L53C28

```
 if (!success) {
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L66C12-L66C28

```
 if (!gasIsEth) {
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L108C8-L108C25

```
    if (!success) {
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L35C5-L35C24

```
     if (!success) {
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L62C4-L62C24

```
      if (!gasCurrencyIsEth || !deliverEth) {
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L77C3-L77C48

## [G-12] Using storage instead of memory for structs/arrays saves gas (Not found by the bot)

```
     function swapExactIn(
        SwapParams memory swapParams, // SwapParams is a struct
        address receiver
    ) public payable routerIsSet returns (uint256 amountOut) {
        swapParams = _receiveAndWrapIfNeeded(swapParams);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L123C3-L127C58


```
   function swapExactOut(
        SwapParams memory swapParams,
        address receiver,
        address refundAddress
    ) public payable routerIsSet returns (uint256 amountIn) {
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L143C3-L147C62

## [G-13] Leverage mapping and dot notation for struct assignment

In dot notation, values are directly written to storage variable, When we use the current method in the code the compiler will allocate some memory to store the struct instance first before writing it to storage.

```
     ICommonOFT.LzCallParams memory callParams = ICommonOFT.LzCallParams({
            refundAddress: payable(msg.sender),
            zroPaymentAddress: address(0x0),
            adapterParams: adapterParams
        });

```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L170C6-L175C1
