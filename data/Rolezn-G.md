## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | Functions guaranteed to revert when called by normal users can be marked `payable` | 4 | 84 |
| [GAS&#x2011;2](#GAS&#x2011;2) | Use shift Right instead of division if possible to save gas | 2 | 40 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 1 | 6 |
| [GAS&#x2011;4](#GAS&#x2011;4) | `abi.encode()` is less efficient than `abi.encodepacked()` | 3 | 159 |
| [GAS&#x2011;5](#GAS&#x2011;5) | Using XOR (^) and AND (&) bitwise equivalents for gas optimizations | 17 | 221 |
| [GAS&#x2011;6](#GAS&#x2011;6) | Avoid zero transfer to save gas | 11 | 1100 |
| [GAS&#x2011;7](#GAS&#x2011;7) | Using fixed bytes is cheaper than using `string` | 1 | 13 |
| [GAS&#x2011;8](#GAS&#x2011;8) | `address(this)` can be stored in `state variable` that will ultimately cost less, than every time calculating this, specifically when it used multiple times in a `contract` | 23 | 1150 |
| [GAS&#x2011;9](#GAS&#x2011;9) | State variables that are used multiple times in a function should be cached in stack variables | 1 | 97 |
| [GAS&#x2011;10](#GAS&#x2011;10) | Empty Blocks Should Be Removed Or Emit Something | 7 | 77 |
| [GAS&#x2011;11](#GAS&#x2011;11) | Cast only once instead of repeatedly casting values to same type | 9 | - |
| [GAS&#x2011;12](#GAS&#x2011;12) | Use assembly in place of `abi.decode` to extract calldata values more efficiently | 12 | - |
| [GAS&#x2011;13](#GAS&#x2011;13) | Add `unchecked \{\}` for subtractions where the operands cannot underflow | 1 | 40 |
| [GAS&#x2011;14](#GAS&#x2011;14) | Use `assembly` to write address storage values | 4 | 296 |
| [GAS&#x2011;15](#GAS&#x2011;15) | Using `this` to access functions results in an external call, wasting gas | 1 | 100 |
| [GAS&#x2011;16](#GAS&#x2011;16) | Hash using `assembly` instead of solidity | 1 | 80 |
| [GAS&#x2011;17](#GAS&#x2011;17) | Use hardcoded address instead `address(this)` | 24 | 624 |

Total: 197 contexts over 17 issues


The following findings include instances that are not present in the automated findings.


## Gas Optimizations

### <a href="#gas-summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: DcntEth.sol

32: function mintByOwner(address _to, uint256 _amount) public onlyOwner {

```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L32

```solidity
File: DcntEth.sol

36: function burnByOwner(address _from, uint256 _amount) public onlyOwner {

```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L36


```solidity
File: UTBFeeCollector.sol

18: function setSigner(address _signer) public onlyOwner {

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBFeeCollector.sol#L18


```solidity
File: UTBFeeCollector.sol

69: function redeemFees(address token, uint amount) public onlyOwner {

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBFeeCollector.sol#L69


</details>



### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Use shift Right instead of division if possible to save gas

#### <ins>Proof Of Concept</ins>


```solidity
File: StargateBridgeAdapter.sol

66: return (amt2Bridge * (1e4 - SG_FEE_BPS)) / 1e4;

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/StargateBridgeAdapter.sol#L66

```solidity
File: StargateBridgeAdapter.sol

176: (amt2Bridge * (10000 - SG_FEE_BPS)) / 10000,

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/StargateBridgeAdapter.sol#L176





### <a href="#gas-summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.23/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

#### <ins>Proof Of Concept</ins>


```solidity
File: SwapParams.sol

14: uint8 direction;
```

https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers/SwapParams.sol#L14






### <a href="#gas-summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> `abi.encode()` is less efficient than `abi.encodepacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison 

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: DecentEthRouter.sol

99: destBridge = bytes32(abi.encode(destinationBridges[_dstChainId]));
```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L99


```solidity
File: UniSwapper.sol

40: return abi.encode(newSwapParams, receiver, refund);
```

https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers/UniSwapper.sol#L40

```solidity
File: StargateBridgeAdapter.sol

81: bridgePayload = abi.encode(
            postBridge,
            target,
            paymentOperator,
            payload,
            refund
        );

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/StargateBridgeAdapter.sol#L81



</details>

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |




### <a href="#gas-summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Using XOR (^) and AND (&) bitwise equivalents for gas optimizations

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: DecentEthRouter.sol

100: if (msgType == MT_ETH_TRANSFER) {

```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L100

```solidity
File: DecentEthRouter.sol

250: if (msgType == MT_ETH_TRANSFER_WITH_PAYLOAD) {

```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L250

```solidity
File: DecentEthRouter.sol

271: if (msgType == MT_ETH_TRANSFER) {

```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L271

```solidity
File: UTB.sol

74: if (swapParams.tokenIn == address(0)) {

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTB.sol#L74

```solidity
File: UTB.sol

97: if (tokenOut == address(0)) {

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTB.sol#L97

```solidity
File: UTB.sol

142: if (tokenOut == address(0)) {

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTB.sol#L142

```solidity
File: UTBFeeCollector.sol

29: require(signature.length == 65, "Invalid signature length");

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBFeeCollector.sol#L29

```solidity
File: UTBFeeCollector.sol

54: require(recovered == signer, "Wrong signature");

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBFeeCollector.sol#L54

```solidity
File: UTBFeeCollector.sol

70: if (token == address(0)) {

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBFeeCollector.sol#L70

```solidity
File: UTBExecutor.sol

51: if (token == address(0)) {

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L51

```solidity
File: UTBExecutor.sol

76: if (remainingBalance == 0) {

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L76

```solidity
File: UniSwapper.sol

52: if (token == address(0)) {

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers/UniSwapper.sol#L52

```solidity
File: UniSwapper.sol

68: if (swapParams.path.length == 0) {

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers/UniSwapper.sol#L68

```solidity
File: UniSwapper.sol

71: if (swapParams.direction == SwapDirection.EXACT_IN) {

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers/UniSwapper.sol#L71

```solidity
File: UniSwapper.sol

107: if (swapParams.direction == SwapDirection.EXACT_OUT) {

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers/UniSwapper.sol#L107

```solidity
File: UniSwapper.sol

115: uint amt2Recipient = swapParams.direction == SwapDirection.EXACT_OUT

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers/UniSwapper.sol#L115

```solidity
File: StargateBridgeAdapter.sol

109: bridgeToken == stargateEth

```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/StargateBridgeAdapter.sol#L109



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |






### <a href="#gas-summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Avoid zero transfer to save gas

In Solidity, unnecessary operations can waste gas. For example, a transfer function without a zero amount check uses gas even if called with a zero amount, since the contract state remains unchanged. Implementing a zero amount check avoids these unnecessary function calls, saving gas and improving efficiency.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: DecentEthRouter.sol

267: dcntEth.transfer(_to, _amount);

273: weth.transfer(_to, _amount);

276: payable(_to).transfer(_amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L267

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L273

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L276



```solidity
File: DecentEthRouter.sol

290: payable(msg.sender).transfer(amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L290


```solidity
File: DecentBridgeExecutor.sol

36: weth.transfer(from, amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L36



```solidity
File: DecentBridgeExecutor.sol

63: payable(from).transfer(amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L63


```solidity
File: UTBFeeCollector.sol

56: IERC20(fees.feeToken).transferFrom(


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBFeeCollector.sol#L56

```solidity
File: UTBFeeCollector.sol

71: payable(owner).transfer(amount);

73: IERC20(token).transfer(owner, amount);


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBFeeCollector.sol#L71

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBFeeCollector.sol#L73



```solidity
File: UTBExecutor.sol

61: IERC20(token).transferFrom(msg.sender, address(this), amount);

80: IERC20(token).transfer(refund, remainingBalance);


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L61

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L80




</details>





### <a href="#gas-summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Using fixed bytes is cheaper than using `string`

As a rule of thumb, use `bytes` for arbitrary-length raw byte data and string for arbitrary-length `string` (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of `bytes1` to `bytes32` because they are much cheaper.

#### <ins>Proof Of Concept</ins>


```solidity
File: UTBFeeCollector.sol

10: string constant BANNER = "/x19Ethereum Signed Message:/n32";
```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBFeeCollector.sol#L10






### <a href="#gas-summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> `address(this)` can be stored in `state variable` that will ultimately cost less, than every time calculating this, specifically when it used multiple times in a `contract`

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: DecentEthRouter.sol

181: weth.transferFrom(msg.sender, address(this), _amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L181

```solidity
File: DecentEthRouter.sol

266: if (weth.balanceOf(address(this)) < _amount) {


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L266

```solidity
File: DecentEthRouter.sol

288: dcntEth.transferFrom(msg.sender, address(this), amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L288

```solidity
File: DecentEthRouter.sol

297: dcntEth.transferFrom(msg.sender, address(this), amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L297

```solidity
File: DecentEthRouter.sol

309: dcntEth.mint(address(this), msg.value);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L309

```solidity
File: DecentEthRouter.sol

316: dcntEth.burn(address(this), amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L316

```solidity
File: DecentEthRouter.sol

325: weth.transferFrom(msg.sender, address(this), amount);

326: dcntEth.mint(address(this), amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L325-L326

```solidity
File: DecentEthRouter.sol

333: dcntEth.burn(address(this), amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L333

```solidity
File: DecentBridgeExecutor.sol

30: uint256 balanceBefore = weth.balanceOf(address(this));

41: (balanceBefore - weth.balanceOf(address(this)));


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L30

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L41



```solidity
File: DecentBridgeExecutor.sol

75: weth.transferFrom(msg.sender, address(this), amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L75

```solidity
File: UTB.sol

85: address(this),


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTB.sol#L85

```solidity
File: UTBFeeCollector.sol

58: address(this),


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBFeeCollector.sol#L58

```solidity
File: UTBExecutor.sol

59: uint initBalance = IERC20(token).balanceOf(address(this));

61: IERC20(token).transferFrom(msg.sender, address(this), amount);

73: uint remainingBalance = IERC20(token).balanceOf(address(this)) -


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L59

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L61

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L73



```solidity
File: UniSwapper.sol

85: address(this),


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers/UniSwapper.sol#L85

```solidity
File: UniSwapper.sol

132: recipient: address(this),


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers/UniSwapper.sol#L132

```solidity
File: UniSwapper.sol

152: recipient: address(this),


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers/UniSwapper.sol#L152

```solidity
File: DecentBridgeAdapter.sol

111: address(this),


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/DecentBridgeAdapter.sol#L111

```solidity
File: DecentBridgeAdapter.sol

141: address(this),


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/DecentBridgeAdapter.sol#L141

```solidity
File: StargateBridgeAdapter.sol

88: IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88



</details>




### <a href="#gas-summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> State variables that are used multiple times in a function should be cached in stack variables

When performing multiple operations on a state variable in a function, it is recommended to cache it first. Either multiple reads or multiple writes to a state variable can save gas by caching it on the stack.
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.
*Saves 100 gas per instance*.

#### <ins>Proof Of Concept</ins>



```solidity
File: DecentBridgeAdapter.sol

/// @audit Affected state variables: router

45: function estimateFees(
        SwapInstructions memory postBridge,
        uint256 dstChainId,
        address target,
        uint64 dstGas,
        bytes memory payload
    ) public view returns (uint nativeFee, uint zroFee) {
        SwapParams memory swapParams = abi.decode(
            postBridge.swapPayload,
            (SwapParams)
        );
        return
            router.estimateSendAndCallFee( // @audit <---
                router.MT_ETH_TRANSFER_WITH_PAYLOAD(), // @audit <---
                lzIdLookup[dstChainId],
                target,
                swapParams.amountIn,
                dstGas,
                false,
                payload
            );
    }


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/DecentBridgeAdapter.sol#L44-L65






### <a href="#gas-summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Empty Blocks Should Be Removed Or Emit Something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: DecentEthRouter.sol

339: fallback() external payable {}
```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L339

```solidity
File: DecentBridgeExecutor.sol

88: fallback() external payable {}
```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L88

```solidity
File: UTB.sol

341: fallback() external payable {}
```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTB.sol#L341

```solidity
File: UTBFeeCollector.sol

79: fallback() external payable {}
```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBFeeCollector.sol#L79

```solidity
File: UniSwapper.sol

173: fallback() external payable {}
```

https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers/UniSwapper.sol#L173

```solidity
File: DecentBridgeAdapter.sol

158: fallback() external payable {}
```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/DecentBridgeAdapter.sol#L158

```solidity
File: StargateBridgeAdapter.sol

220: fallback() external payable {}
```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/StargateBridgeAdapter.sol#L220



</details>




### <a href="#gas-summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Cast only once instead of repeatedly casting values to same type

`IERC20()` should be cast only once and result should be cached.

#### <ins>Proof Of Concept</ins>


```solidity
File: UTBExecutor.sol

59: uint initBalance = IERC20(token).balanceOf(address(this));

61: IERC20(token).transferFrom(msg.sender, address(this), amount);

62: IERC20(token).approve(paymentOperator, amount);

73: uint remainingBalance = IERC20(token).balanceOf(address(this)) -

80: IERC20(token).transfer(refund, remainingBalance);


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L59

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L61

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L62

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L73

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L80



```solidity
File: DecentBridgeAdapter.sol

109: IERC20(bridgeToken).transferFrom(

114: IERC20(bridgeToken).approve(address(router), amt2Bridge);


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/DecentBridgeAdapter.sol#L109

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/DecentBridgeAdapter.sol#L114



```solidity
File: StargateBridgeAdapter.sol

88: IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);

89: IERC20(bridgeToken).approve(address(router), amt2Bridge);


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88-L89



### <a href="#gas-summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> Use assembly in place of `abi.decode` to extract calldata values more efficiently

Instead of using `abi.decode` we can use assembly to decode our desired `calldata` values directly. This will allow us to avoid decoding `calldata` values that we will not use. we can use assembly to decode our desired `calldata` values directly. This will allow us to avoid decoding `calldata` values that we will not use

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: DecentEthRouter.sol

251: (, , , , callPayload) = abi.decode(
                _payload,
                (uint8, address, address, bool, bytes)
            );


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L251

```solidity
File: UTB.sol

69: SwapParams memory swapParams = abi.decode(
            swapInstructions.swapPayload,
            (SwapParams)
        );


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTB.sol#L69

```solidity
File: UTB.sol

181: SwapParams memory newPostSwapParams = abi.decode(
            instructions.postBridge.swapPayload,
            (SwapParams)
        );


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTB.sol#L181

```solidity
File: DecentBridgeAdapter.sol

51: SwapParams memory swapParams = abi.decode(
            postBridge.swapPayload,
            (SwapParams)
        );


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/DecentBridgeAdapter.sol#L51

```solidity
File: DecentBridgeAdapter.sol

96: uint64 dstGas = abi.decode(additionalArgs, (uint64));

103: SwapParams memory swapParams = abi.decode(
            postBridge.swapPayload,
            (SwapParams)
        );


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/DecentBridgeAdapter.sol#L96

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/DecentBridgeAdapter.sol#L103



```solidity
File: DecentBridgeAdapter.sol

134: SwapParams memory swapParams = abi.decode(
            postBridge.swapPayload,
            (SwapParams)
        );


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/DecentBridgeAdapter.sol#L134

```solidity
File: StargateBridgeAdapter.sol

58: bridgeToken = abi.decode(additionalArgs, (address));


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/StargateBridgeAdapter.sol#L58

```solidity
File: StargateBridgeAdapter.sol

79: address bridgeToken = abi.decode(additionalArgs, (address));


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/StargateBridgeAdapter.sol#L79

```solidity
File: StargateBridgeAdapter.sol

104: (address bridgeToken, LzBridgeData memory lzBridgeData) = abi.decode(
            additionalArgs,
            (address, LzBridgeData)
        );


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/StargateBridgeAdapter.sol#L104

```solidity
File: StargateBridgeAdapter.sol

197: ) = abi.decode(
                payload,
                (SwapInstructions, address, address, bytes, address)
            );

202: SwapParams memory swapParams = abi.decode(
            postBridge.swapPayload,
            (SwapParams)
        );


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/StargateBridgeAdapter.sol#L197

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/StargateBridgeAdapter.sol#L202





</details>




### <a href="#gas-summary">[GAS&#x2011;13]</a><a name="GAS&#x2011;13"> Add `unchecked /{/}` for subtractions where the operands cannot underflow

#### <ins>Proof Of Concept</ins>


```solidity
File: DecentEthRouter.sol

179: gasValue = msg.value - _amount;

```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L179




### <a href="#gas-summary">[GAS&#x2011;14]</a><a name="GAS&#x2011;14"> Use `assembly` to write address storage values [SAFE]

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: DcntEth.sol

21: router = _router;
```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L21

```solidity
File: UTBFeeCollector.sol

19: signer = _signer;
```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBFeeCollector.sol#L19

```solidity
File: DecentBridgeAdapter.sol

23: bridgeToken = _bridgeToken;
```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/DecentBridgeAdapter.sol#L23

```solidity
File: StargateBridgeAdapter.sol

38: stargateEth = _sgEth;
```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/StargateBridgeAdapter.sol#L38



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
    
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
    
        function testGas() public {
            c0.setOwnerAssembly(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
            c1.setOwner(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
        }
    }
    
    contract Contract0 {
    
        address owner;
        function setOwnerAssembly(address _owner) public {
            assembly{
                sstore(owner.slot,_owner)
            }
        }
    
    }
    
    contract Contract1 {
        address owner;
        function setOwner(address _owner) public {
            owner = _owner;
        }
    
    }

#### Gas Report

|  Contract0 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 35287                                     | 207             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwnerAssembly                          | 22324           | 22324 | 22324  | 22324 | 1       |


|  Contract1 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 48499                                     | 273             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwner                                  | 22363           | 22363 | 22363  | 22363 | 1       |




### <a href="#gas-summary">[GAS&#x2011;15]</a><a name="GAS&#x2011;15"> Using `this` to access functions results in an external call, wasting gas

External calls have an overhead of **100 gas**, which can be avoided by not referencing the function using `this`. Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents' functions and change the visibility from `external` to `public`, so make this change if it's required in order to call the function internally.

#### <ins>Proof Of Concept</ins>

```solidity
File: DecentBridgeAdapter.sol

99: this.receiveFromBridge,


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/DecentBridgeAdapter.sol#L99






### <a href="#gas-summary">[GAS&#x2011;16]</a><a name="GAS&#x2011;16"> Hash using `assembly` instead of solidity

Saves about ~80 gas

```solidity
function solidityHash(uint256 a, uint256 b) public view {
    //unoptimized
    keccak256(abi.encodePacked(a, b));
}
```
Gas Test: 313

```solidity
function assemblyHash(uint256 a, uint256 b) public view {
    //optimized
    assembly {
        mstore(0x00, a)
        mstore(0x20, b)
        let hashedVal := keccak256(0x00, 0x40)
    }
} 
```
Gas Test: 231

#### <ins>Proof Of Concept</ins>


```solidity
File: UTBFeeCollector.sol

49: bytes32 constructedHash = keccak256(
            abi.encodePacked(BANNER, keccak256(packedInfo))
        );


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBFeeCollector.sol#L49






### <a href="#gas-summary">[GAS&#x2011;17]</a><a name="GAS&#x2011;17"> Use hardcoded address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry's script.sol and solmate's `LibRlp.sol` contracts can help achieve this.

References: 
https://book.getfoundry.sh/reference/forge-std/compute-create-address 

https://twitter.com/transmissions11/status/1518507047943245824

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: DecentEthRouter.sol

181: weth.transferFrom(msg.sender, address(this), _amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L181


```solidity
File: DecentEthRouter.sol

266: if (weth.balanceOf(address(this)) < _amount) {


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L266

```solidity
File: DecentEthRouter.sol

288: dcntEth.transferFrom(msg.sender, address(this), amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L288

```solidity
File: DecentEthRouter.sol

297: dcntEth.transferFrom(msg.sender, address(this), amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L297

```solidity
File: DecentEthRouter.sol

309: dcntEth.mint(address(this), msg.value);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L309

```solidity
File: DecentEthRouter.sol

316: dcntEth.burn(address(this), amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L316

```solidity
File: DecentEthRouter.sol

325: weth.transferFrom(msg.sender, address(this), amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L325

```solidity
File: DecentEthRouter.sol

333: dcntEth.burn(address(this), amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L333

```solidity
File: DecentBridgeExecutor.sol

30: uint256 balanceBefore = weth.balanceOf(address(this));

41: (balanceBefore - weth.balanceOf(address(this)));


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L30

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L41



```solidity
File: DecentBridgeExecutor.sol

75: weth.transferFrom(msg.sender, address(this), amount);


```

https://github.com/decentxyz/decent-bridge/tree/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L75

```solidity
File: UTB.sol

85: address(this),


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTB.sol#L85

```solidity
File: UTBFeeCollector.sol

58: address(this),


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBFeeCollector.sol#L58

```solidity
File: UTBExecutor.sol

59: uint initBalance = IERC20(token).balanceOf(address(this));

61: IERC20(token).transferFrom(msg.sender, address(this), amount);

73: uint remainingBalance = IERC20(token).balanceOf(address(this)) -


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L59

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L61

https://github.com/code-423n4/2024-01-decent/tree/main/src/UTBExecutor.sol#L73



```solidity
File: UniSwapper.sol

85: address(this),


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers/UniSwapper.sol#L85

```solidity
File: UniSwapper.sol

132: recipient: address(this),


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers/UniSwapper.sol#L132

```solidity
File: UniSwapper.sol

152: recipient: address(this),


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/swappers/UniSwapper.sol#L152

```solidity
File: DecentBridgeAdapter.sol

111: address(this),


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/DecentBridgeAdapter.sol#L111

```solidity
File: DecentBridgeAdapter.sol

141: address(this),


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/DecentBridgeAdapter.sol#L141

```solidity
File: StargateBridgeAdapter.sol

88: IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);


```

https://github.com/code-423n4/2024-01-decent/tree/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88



</details>

#### <ins>Recommended Mitigation Steps</ins>

Use hardcoded `address`

