## Summary

### Gas Optimization

no |Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Avoid calling the same internal function multiple times |  |20|--| 
| [G-02] |Remove duplicate reference to randomizer contract |  |7|--| 
| [G-03] |public functions not called by the contract should be declared external instead |  |12|--| 
| [G-04] |Duplicated require()/if() checks should be refactored to a modifier or function |  |13|--| 
| [G-05] | x += y costs more gas than x = x + y for state variables |  |1|--| 
| [G-06] |Using a positive conditional flow to save a NOT opcode |  |8|--| 
| [G-07] |Not using the named return variables when a function returns, wastes deployment gas |  |3|--| 
| [G-08] |Use assembly for math (add, sub, mul, div)  | |3|--| 




## Gas Optimizations  

## [G-1] Avoid calling the same internal function multiple times
Calling an internal function will cost at least ~16-20 gas (JUMP to internal function instructions and JUMP back to original instructions). If you call the same internal function multiple times you can save gas by caching the return value of the internal function. 

```solidity

file:/src/UTB.sol#L134-L16
 134    function _swapAndExecute(
135     SwapInstructions memory swapInstructions,
136        address target,
137        address paymentOperator,
138       bytes memory payload,
139        address payable refund
140   ) private {
141        (address tokenOut, uint256 amountOut) = performSwap(swapInstructions);
142        if (tokenOut == address(0)) {
143  ..         executor.execute{value: amountOut}(
144                target,
145                paymentOperator,
146                payload,
147                tokenOut,
148               amountOut,
150                refund
            );
153     } else {
154         IERC20(tokenOut).approve(address(executor), amountOut);
155         executor.execute(
156            target,
157           paymentOperator,
                payload,
                tokenOut,
                amountOut,
                refund
            );

```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L134-L164
```solidity

File: src/DecentEthRouter.sol
 225       _bridgeWithPayload(
 294        function redeemWeth(
322      function addLiquidityWeth(
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L225-L225


## [G-2]Remove duplicate reference to randomizer contract
t is sufficient to only store the address of the randomizer in SwapParams.  The randomizer contract instance can be referenced by providing this address . The gas savings also cascade to critical operations such as mint , burnToMint, and airDropTokens. 

STEP 1: Remove duplicate declaration from struct:
```solidity
file: 
 10  struct SwapParams {
 11   uint256 amountIn;
 12   uint256 amountOut;
 13   address tokenIn;
 14   address tokenOut;
 15   uint8 direction;
 16   bytes path;
```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/swappers/SwapParams.sol#L9

## [G-3]  public functions not called by the contract should be declared external instead
 when a function is declared as public, it is generated with an internal and an external interface. This means the function can be called both internally (within the contract) and externally (by other contracts or accounts). However, if a public function is never called internally and is only expected to be invoked externally, it is more gas-efficient to explicitly declare it as external. 

```solidity
file:src/UTB.sol
 28   function setExecutor(address _executor) public onlyOwner {
     
 44   function setFeeCollector(address payable _feeCollector) public onlyOwner {

259      function bridgeAndExecute(
311     function receiveFromBridge(
325       function registerSwapper(address swapper) public onlyOwner {
334         function registerBridge(address bridge) public onlyOwner {

```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L28
```soloidity

file:
19    function execute(
41       function execute(
```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBExecutor.sol#L19-L19
```solidity
file:src/UTBFeeCollector.sol 
18  function setSigner(address _signer) public onlyOwner { 
44       function collectFees(
69       function redeemFees(address token, uint amount) public onlyOwner {
```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBFeeCollector.sol#L18
```solidity
file:
19    function setBridgeExecutor(address _executor) public onlyOwner {
  
```
  https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/bridge_adapters/BaseAdapter.sol#L19
## [G-4] Duplicated require()/if() checks should be refactored to a modifier or function
It is common to use require() or if() statements to validate certain conditions before executing specific code. However, when the same checks are repeated multiple times within a contract, it can result in redundant code and unnecessary gas consumption.
To save gas and improve code readability and maintainability, it is recommended to refactor duplicated checks into modifiers or functions. By doing so, the checks can be abstracted into reusable code blocks that can be applied to multiple functions within the contract.

```solidity
file:src/UTB.so
97        if (tokenOut == address(0)) {
215          if (bridgeToken != address(0)) {

```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L97-L97

```solidity
file:src/UTBExecutor.sol
51         if (token == address(0)) {
53          if (!success) {
66           if (!success) {
76         if (remainingBalance == 0) {
```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBExecutor.sol#L51

https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBExecutor.sol#L53

https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBExecutor.sol#L66

https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBExecutor.sol#L76
```solidity
file:src/bridge_adapters/DecentBridgeAdapter.sol
 108       if (!gasIsEth) {

```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/bridge_adapters/DecentBridgeAdapter.sol#L108

```solidity
file:src/swappers/UniSwapper.sol
 52       if (token == address(0)) {
 68         if (swapParams.path.length == 0) {
```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/swappers/UniSwapper.sol#L52
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/swappers/UniSwapper.sol#L68
```slidity

file:src/DecentEthRouter.sol
 177       if (gasCurrencyIsEth) {
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L177
```solidity
file:src/DecentBridgeExecutor.sol
 35       if (!success) {
 62         if (!success) {
 77           if (!gasCurrencyIsEth || !deliverEth) {  
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L35

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L62

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L77
## [G-5]  x += y costs more gas than x = x + y for state variables

```solidity
file:src/DecentEthRouter.sol
  balanceOf[msg.sender] += amount;

```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L56C8-L56C41
       

## [G-6] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidity
file:src/UTB.sol
215          if (bridgeToken != address(0)) {
235            if (fees.feeToken != address(0)) {
```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L215
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTB.sol#L235
```solidity
file:/src/UTBExecutor.sol
 53           if (!success) {
 66            if (!success) {
```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBExecutor.sol#L53

```solidity
file:src/UTBFeeCollector.sol
  55      if (fees.feeToken != address(0)) {

```

https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBFeeCollector.sol#L55
```solidity
file:src/bridge_adapters/DecentBridgeAdapter.sol
108       if (!gasIsEth) {

```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/bridge_adapters/DecentBridgeAdapter.sol#L108

```solidity
file:src/swappers/UniSwapper.sol
 82       if (swapParams.tokenIn != address(0)) {
```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/swappers/UniSwapper.sol#L82
```solidity
file:src/DecentBridgeExecutor.sol
35      if(!sucess){
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L35
## [G-7] Not using the named return variables when a function returns, wastes deployment gas

```solidity
file:src/bridge_adapters/DecentBridgeAdapter.sol
78     return amt2Bridge;
## Recommended Code
    return (uint40(block.timestamp) - lastTimestamp);
        
```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/bridge_adapters/DecentBridgeAdapter.sol#L78C2-L78C27

```solidity
file:/src/bridge_adapters/StargateBridgeAdapter
108      return bridgeToken == stargateEth
             ? (lzBridgeData.fee + amt2Bridge)
                : lzBridgeData.fee;bridgeToken == stargateEth
             ? (lzBridgeData.fee + amt2Bridge)
                : lzBridgeData.fee
 131       return lzBridgeData._dstChainId;
```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/bridge_adapters/StargateBridgeAdapter.sol#L108-L111

https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/bridge_adapters/StargateBridgeAdapter.sol#L131


## [G-8]Use assembly for math (add, sub, mul, div)
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.
```solidity
file:src/UTBExecutor
 73       uint remainingBalance = IERC20(token).balanceOf(address(this)) - initBalance

```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBExecutor.sol#L73

```solidity
file:
66       return (amt2Bridge * (1e4 - SG_FEE_BPS)) / 1e4;
176   (amt2Bridge * (10000 - SG_FEE_BPS)) / 10000, // the min qty you would accept on the destination,           fee  is 6 bips   
```
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/bridge_adapters/StargateBridgeAdapter.sol#L66

https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/bridge_adapters/StargateBridgeAdapter.sol#L176
