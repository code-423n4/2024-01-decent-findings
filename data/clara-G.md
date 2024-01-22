# GAS OPTIMIZATION

# SUMMARY
|      |  issue  |  instance  |
|------|---------|------------|
|[G‑01]|Gas Optimization: Change Public State Variable Visibility to Private|13|
|[G‑02]|Gas Optimization: Avoiding address(this) in Favor of Hardcoded Addresses|20|
|[G‑03]|Optimize External Calls with Assembly for Memory Efficiency|16|
|[G‑04]|Gas Optimization: Use Assembly for Efficient Hashing|1|
|[G‑05]|Gas Optimization Issue - Redundant Stack Variable Assignment|1|
|[G‑06]|Gas Optimization Issue - Gas-Efficient Alternatives for Common Math Operations|2|
|[G‑07]|Using a positive conditional flow to save a NOT opcode|1|
|[G‑08]|Gas Optimization: Using Assembly for Efficient Back-to-Back External Calls|4|
|[G‑09]|Gas Optimization: Favor Ternary Operation Over If-Else Statement|2|
|[G‑10]| Unlimited gas consumption risk due to external call recipients|3|
|[G‑11]|Gas Optimization: Prefer `bytes.concat()` Over `abi.encodePacked` for Non-Hashing Concatenation|1|
|[G‑12]|Gas Optimization: Prefer Low-Level Calls Over Contract Existence Checks|21|
|[G‑13]|Sort Solidity operations using short-circuit mode|2|
|[G‑14]|Gas Optimization: Prefer Storage over Memory for Structs/Arrays|3|
|[G‑15]|bytes constants are more efficient than string constants|1|






## [G-01] Gas Optimization: Change Public State Variable Visibility to Private

**Explanation of Issue:**
The current state variables in the contract have public visibility, which exposes them to potential security risks, increases complexity, and incurs higher gas costs. It is recommended to change the visibility to private unless there is a specific need for public access.




There are 13 instance(s) of this issue:
```solidity
File: src/DcntEth.sol
6  address public router;
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L6

```solidity
File: src/DecentBridgeExecutor.sol
10   bool public gasCurrencyIsEth; // for chains that use ETH as gas currency
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L10

```solidity
File: src/DecentEthRouter.sol
    IWETH public weth;
    IDcntEth public dcntEth;
    IDecentBridgeExecutor public executor;

    uint8 public constant MT_ETH_TRANSFER = 0;
    uint8 public constant MT_ETH_TRANSFER_WITH_PAYLOAD = 1;

    uint16 public constant PT_SEND_AND_CALL = 1;

    bool public gasCurrencyIsEth; // for chains that use ETH as gas currency
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L13-L22


```solidity
File: src/bridge_adapters/BaseAdapter.sol
7  address public bridgeExecutor;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L7


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol
24   address public stargateEth;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L24


```solidity
File: src/swappers/UniSwapper.sol
17    address public uniswap_router;
18    address payable public wrapped;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L17-L18


## [G-02] Avoiding address(this) in Favor of Hardcoded Addresses

**Explanation of Issue:**

In certain scenarios, opting for a hardcoded address over the use of `address(this)` can lead to more gas-efficient smart contracts. This recommendation is particularly relevant when the same contract address is required multiple times within the contract code.



There are 20 instance(s) of this issue:
```solidity
File: src/DecentBridgeExecutor.sol
30  uint256 balanceBefore = weth.balanceOf(address(this));

41  (balanceBefore - weth.balanceOf(address(this)));

75  weth.transferFrom(msg.sender, address(this), amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L30



```solidity
File: src/DecentEthRouter.sol
51   require(weth.balanceOf(address(this)) > amount, "not enough reserves");

181  weth.transferFrom(msg.sender, address(this), _amount);

186  address(this), // from address that has dcntEth (so DecentRouter)

186  if (weth.balanceOf(address(this)) < _amount) {

288  dcntEth.transferFrom(msg.sender, address(this), amount);

297   dcntEth.transferFrom(msg.sender, address(this), amount);

309   dcntEth.mint(address(this), msg.value);

316   dcntEth.burn(address(this), amount);

325   weth.transferFrom(msg.sender, address(this), amount);
326   dcntEth.mint(address(this), amount);

333   dcntEth.burn(address(this), amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L51



```solidity
File: src/UTBExecutor.sol
59  uint initBalance = IERC20(token).balanceOf(address(this));

61  IERC20(token).transferFrom(msg.sender, address(this), amount);

73  uint remainingBalance = IERC20(token).balanceOf(address(this)) -
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L59


```solidity
File: src/swappers/UniSwapper.sol
85    address(this),

132   recipient: address(this),

152   recipient: address(this),
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L85






## [G-03] Optimize External Calls with Assembly for Memory Efficiency

**Explanation of Issue:** Using interfaces to make external contract calls in Solidity can be inefficient in terms of memory utilization. Each such call involves creating a new memory location to store the data being passed, resulting in memory expansion costs. This can impact gas consumption, especially for contracts that make frequent external calls. However, using inline assembly allows for optimized memory usage by reusing already allocated memory spaces or utilizing scratch space for smaller datasets. This optimization can lead to notable gas savings. Furthermore, inline assembly enables important safety checks, such as verifying if the target address has code deployed to it using `extcodesize(addr)` before making the call, mitigating risks associated with contract interactions.


There are 16 instances of this issue:
```solidity
File:  src/DecentBridgeExecutor.sol
30        uint256 balanceBefore = weth.balanceOf(address(this));
31        weth.approve(target, amount);

36   weth.transfer(from, amount);

44   weth.transfer(from, remainingAfterCall);

60   weth.withdraw(amount);

75   weth.transferFrom(msg.sender, address(this), amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L


```solidity
File: src/swappers/UniSwapper.sol
137         IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
138         amountOut = IV3SwapRouter(uniswap_router).exactInput(params);

158        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
159        amountIn = IV3SwapRouter(uniswap_router).exactOutput(params);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L137

```solidity
File: src/UTB.sol
83   IERC20(swapParams.tokenIn).transferFrom(

90   IERC20(swapParams.tokenIn).approve(

152  IERC20(tokenOut).approve(address(executor), amountOut);

211  IBridgeAdapter bridgeAdapter = IBridgeAdapter(bridgeAdapters[instructions.bridgeId]);

216  IERC20(bridgeToken).approve(address(bridgeAdapter), amt2Bridge);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L83


```solidity
File: src/UTBFeeCollector.sol
73  IERC20(token).transfer(owner, amount);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L73






## [G-04] Gas Optimization: Use Assembly for Efficient Hashing

**Explanation of Issue:**

When hashing values, employing assembly instead of Solidity can lead to significant gas savings. This optimization is particularly evident when using the keccak256 function.



There are 1 instances of this issue:
```solidity
File: src/UTBFeeCollector.sol
49      bytes32 constructedHash = keccak256(
            abi.encodePacked(BANNER, keccak256(packedInfo))
        );
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L49-L51

## [G‑05] Gas Optimization Issue - Redundant Stack Variable Assignment

**Explanation of Issue:**
When a stack variable is employed as a transient cache for a state variable, and this stack variable is only accessed once, it leads to unnecessary gas consumption. In such cases, it is more gas-efficient to use the state variable directly, avoiding the additional gas cost associated with the redundant stack assignment.


There are 1 instances of this issue:

the `balance` is use only once in the `userIsWithdrawing` modifier.
```solidity
File: src/DecentEthRouter.sol
61  uint256 balance = balanceOf[msg.sender];
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L61

Fix code:
```diff
    modifier userIsWithdrawing(uint256 amount) {
-       uint256 balance = balanceOf[msg.sender];
-       require(balance >= amount, "not enough balance");
+       require(balanceOf[msg.sender] >= amount, "not enough balance");
        _;
        balanceOf[msg.sender] -= amount;
    }    
```


## [G-06] Gas Optimization Issue - Gas-Efficient Alternatives for Common Math Operations

**Explanation of Issue:**
Common math operations, such as min and max, may have more gas-efficient alternatives. In scenarios where unoptimized code uses conditional operators, like the ternary operator, the resulting conditional jumps in opcodes can incur higher gas costs. Exploring gas-efficient alternatives for these operations is crucial for optimizing smart contract performance.


There are 2 instance(s) of this issue:

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol
109          bridgeToken == stargateEth
                ? (lzBridgeData.fee + amt2Bridge)
                : lzBridgeData.fee;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L109-L111


```solidity
File: src/swappers/UniSwapper.sol
115     uint amt2Recipient = swapParams.direction == SwapDirection.EXACT_OUT
            ? swapParams.amountOut
            : swapParams.amountIn;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L115-L117




## [G-07] Using a positive conditional flow to save a NOT opcode

**Explanation of Issue:** By structuring conditional flows in a positive manner, you can save gas by avoiding the use of the NOT opcode.

There are 1 instance(s) of this issue:
```solidity
File: src/DecentBridgeExecutor.sol
77      if (!gasCurrencyIsEth || !deliverEth) {
            _executeWeth(from, target, amount, callPayload);
        } else {
            _executeEth(from, target, amount, callPayload);
        }
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L77-L81

Fix code:
```diff
-       if (!gasCurrencyIsEth || !deliverEth) {
+       if (gasCurrencyIsEth || deliverEth) {
-           _executeWeth(from, target, amount, callPayload);
+           _executeEth(from, target, amount, callPayload);
        } else {
-           _executeEth(from, target, amount, callPayload);
+           _executeWeth(from, target, amount, callPayload);
        }
```




## [G-08] Gas Optimization: Using Assembly for Efficient Back-to-Back External Calls

**Explanation of Issue:**

Performing consecutive external calls with similar function signatures can result in redundant operations and increased gas costs. The use of assembly provides an opportunity to optimize gas consumption by reusing function signatures, parameters, and memory space.



There are 4 instance(s) of this issue:
```solidity
File: src/DecentBridgeExecutor.sol
30        uint256 balanceBefore = weth.balanceOf(address(this));
31         weth.approve(target, amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L30-L31


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol
139     IERC20(swapParams.tokenIn).transferFrom(
            msg.sender,
            address(this),
            swapParams.amountIn
        );

        IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn); 
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L139-L145


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol
88      IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);
        IERC20(bridgeToken).approve(address(router), amt2Bridge); 
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88-L89

```solidity
File: src/UTBExecutor.sol
61        IERC20(token).transferFrom(msg.sender, address(this), amount);
62        IERC20(token).approve(paymentOperator, amount);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L61-L62




## [G-09] Gas Optimization: Favor Ternary Operation Over If-Else Statement

**Explanation of Issue:**

Using a ternary operation instead of an if-else statement can result in reduced gas costs, making it a more gas-efficient choice for conditional assignments.



There are 2 instance(s) of this issue:
```solidity
File: src/DecentBridgeExecutor.sol
        if (!gasCurrencyIsEth || !deliverEth) {
            _executeWeth(from, target, amount, callPayload);
        } else {
            _executeEth(from, target, amount, callPayload);
        }
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L77-L81


```solidity
File: src/UTBFeeCollector.sol
        if (token == address(0)) {
            payable(owner).transfer(amount);
        } else {
            IERC20(token).transfer(owner, amount);
        }
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L70-L74



## [G-10] Unlimited gas consumption risk due to external call recipients

**Explanation of Issue:** When calling an external function without specifying a gas limit, the called contract may consume all the remaining gas, causing the transaction to be reverted. To mitigate this risk, it is recommended to explicitly set a gas limit when making low-level external calls.



There are 3 instances of this issue:

```solidity
File: src/DecentBridgeExecutor.sol
61  (bool success, ) = target.call{value: amount}(callPayload);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L61


```solidity
File: src/UTBExecutor.sol
52   (success, ) = target.call{value: amount}(payload);

65   (success, ) = target.call{value: extraNative}(payload);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L52

## [G-11] Gas Optimization: Prefer `bytes.concat()` Over `abi.encodePacked` for Non-Hashing Concatenation

**Explanation of Issue:**

When concatenating bytes and not intending to use the result for hashing, `bytes.concat()` is a more gas-efficient alternative to `abi.encodePacked`. Using `bytes.concat()` can lead to reduced gas costs in scenarios where concatenation is performed on bytes variables.



There are 1 instances of this issue:
```solidity
File: src/UTBFeeCollector.sol
        bytes32 constructedHash = keccak256(
            abi.encodePacked(BANNER, keccak256(packedInfo))
        );
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L49-L51







## [G-12] Gas Optimization: Prefer Low-Level Calls Over Contract Existence Checks

**Explanation of Issue:**

In Solidity versions prior to 0.8.10, the compiler inserted additional code, including the `EXTCODESIZE` operation (costing 100 gas), to check for the existence of a contract before making an external function call. However, in more recent Solidity versions, these existence checks are skipped if the external call has a return value. To achieve similar gas-efficient behavior in earlier versions, low-level calls can be used since they inherently bypass contract existence checks.




There are 21 instances of this issue:

- Total Save gas : 21 * 100 = `2100`
```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol
109   IERC20(bridgeToken).transferFrom(

114   IERC20(bridgeToken).approve(address(router), amt2Bridge);   

139   IERC20(swapParams.tokenIn).transferFrom(

145   IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn);

147   IUTB(utb).receiveFromBridge(
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L109


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol
88        IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);
89        IERC20(bridgeToken).approve(address(router), amt2Bridge);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88-L89


```solidity
File: src/swappers/UniSwapper.sol
44  IERC20(token).transfer(user, amount);

55  IERC20(token).transfer(recipient, amount);

83  IERC20(swapParams.tokenIn).transferFrom(

91  IWETH(wrapped).deposit{value: swapParams.amountIn}();

137 IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
138 amountOut = IV3SwapRouter(uniswap_router).exactInput(params)

158 IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
159 amountIn = IV3SwapRouter(uniswap_router).exactOutput(params);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L44


```solidity
File: src/UTB.sol
152  IERC20(tokenOut).approve(address(executor), amountOut);

216  IERC20(bridgeToken).approve(address(bridgeAdapter), amt2Bridge);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L152


```solidity
File: src/UTBExecutor.sol
61        IERC20(token).transferFrom(msg.sender, address(this), amount);
62        IERC20(token).approve(paymentOperator, amount);

73   uint remainingBalance = IERC20(token).balanceOf(address(this)) -

80   IERC20(token).transfer(refund, remainingBalance);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L61





## [G-13] Sort Solidity operations using short-circuit mode

**Explanation of Issue:**
The issue involves optimizing the sequencing of Solidity operations using short-circuit mode. Short-circuiting utilizes OR/AND logic to arrange operations based on their gas costs. By placing low gas cost operations at the front and high gas cost operations at the back, subsequent high-cost Ethereum virtual machine operations can be skipped (short-circuited) if the initial low-cost operation evaluates to true.




There are 2 instances of this issue:

- Accessing the state variable `gasCurrencyIsEth` consumes more gas compared to accessing the function parameter `deliverEth`.
```solidity
File: src/DecentBridgeExecutor.sol
77   if (!gasCurrencyIsEth || !deliverEth) {
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L77


```solidity
File: src/DecentEthRouter.sol
272   if (!gasCurrencyIsEth || !deliverEth) {
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L272





## [G-14] Gas Optimization: Prefer Storage over Memory for Structs/Arrays

**Explanation of Issue:**

When dealing with structs or arrays and fetching data from a storage location, opting for memory variables can result in higher gas costs. This is because assigning the data to a memory variable causes all fields of the struct or array to be read from storage, incurring a Gcoldsload (2100 gas) for each field.

Instead of declaring the variable with the memory keyword, it is more gas-efficient to declare it with the storage keyword. Additionally, caching any fields that need to be re-read in stack variables can further optimize gas usage. This approach ensures that only the fields actually read incur the Gcoldsload, resulting in substantial gas savings.




There are 3 instances of this issue:
```solidity
File: src/DecentEthRouter.sol
170   ICommonOFT.LzCallParams memory callParams = ICommonOFT.LzCallParams({
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L170


```solidity
File: src/swappers/UniSwapper.sol
129   IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter

149   IV3SwapRouter.ExactOutputParams memory params = IV3SwapRouter
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L129

## [G-15] bytes constants are more efficient than string constants

**Explanation of Issue:** If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less expensive in terms of gas consumption.



There are 1 instances of this issue:
```solidity
File: src/UTBFeeCollector.sol
10   string constant BANNER = "\x19Ethereum Signed Message:\n32";
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L10

