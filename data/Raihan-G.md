# Gas Optimization

# Sammary

- ### [[G-01] Gas Optimization: Integration of approveAndCheckIfNative Logic into callBridge function](#g-01-gas-optimization-integration-of-approveandcheckifnative-logic-into-callbridge-function)
- ### [[G-02] Gas Optimization: Prefer Low-Level Calls Over Contract Existence Checks](#g-02-gas-optimization-prefer-low-level-calls-over-contract-existence-checks)
- ### [[G-03] Gas Optimization: Change Public State Variable Visibility to Private](#g-03-gas-optimization-change-public-state-variable-visibility-to-private)
- ### [[G-04] Gas Optimization: Avoiding address(this) in Favor of Hardcoded Addresses](#g-04-gas-optimization-avoiding-addressthis-in-favor-of-hardcoded-addresses)
- ### [[G-05] Gas Optimization: Prefer Storage over Memory for Structs/Arrays](#g-05-gas-optimization-prefer-storage-over-memory-for-structsarrays)
- ### [[G-06] bytes constants are more efficient than string constants](#g-06-bytes-constants-are-more-efficient-than-string-constants)
- ### [[G-07] Gas Optimization: Use Assembly for Efficient Hashing](#g-07-gas-optimization-use-assembly-for-efficient-hashing)
- ### [[G‑08] Gas Optimization Issue - Redundant Stack Variable Assignment]()
- ### [[G-09]Gas Optimization Issue - Gas-Efficient Alternatives for Common Math Operations](#g-09gas-optimization-issue---gas-efficient-alternatives-for-common-math-operations)
- ### [[G-10] Using a positive conditional flow to save a NOT opcode](#g-10-using-a-positive-conditional-flow-to-save-a-not-opcode)
- ### [[G-11] Gas Optimization: Using Assembly for Efficient Back-to-Back External Calls](#g-11-gas-optimization-using-assembly-for-efficient-back-to-back-external-calls)
- ### [[G-12] Title: Gas Optimization Issue in External/Internal Function Calls](#g-12-title-gas-optimization-issue-in-externalinternal-function-calls)
- ### [[G-13] Gas Optimization: Favor Ternary Operation Over If-Else Statement](#g-13-gas-optimization-favor-ternary-operation-over-if-else-statement)
- ### [[G-14] Unlimited gas consumption risk due to external call recipients](#g-14-unlimited-gas-consumption-risk-due-to-external-call-recipients)
- ### [[G-15] Gas Optimization: Prefer `bytes.concat()` Over `abi.encodePacked` for Non-Hashing Concatenation](#g-15-gas-optimization-prefer-bytesconcat-over-abiencodepacked-for-non-hashing-concatenation)
- ### [[G-16] Optimize External Calls with Assembly for Memory Efficiency](#g-16-optimize-external-calls-with-assembly-for-memory-efficiency)
- ### [[G-17] Sort Solidity operations using short-circuit mode](#g-17-sort-solidity-operations-using-short-circuit-mode)
- ### [[G-18] Gas Optimization: Use Assembly To Check For address(0)](#g-18-gas-optimization-use-assembly-to-check-for-address0)
- ### [[G-19] Gas Optimization: Use Assembly in Place of `abi.decode` to Extract Calldata Values More Efficiently](#g-19-gas-optimization-use-assembly-in-place-of-abidecode-to-extract-calldata-values-more-efficiently)
- ### [[G-20] Calldata should be used in place of memory function parameters when not mutated](#g-20-calldata-should-be-used-in-place-of-memory-function-parameters-when-not-mutated)
- ### [[G-21] Use custom errors rather than revert()/require() strings to save gas](#g-21-use-custom-errors-rather-than-revertrequire-strings-to-save-gas)
- ### [[G-22] Gas Optimization: Marking Constructors as Payable](#g-22-gas-optimization-marking-constructors-as-payable)
- ### [[G-23] Avoid updating storage when the value hasn't changed to save gas](#g-23-avoid-updating-storage-when-the-value-hasnt-changed-to-save-gas)
- ### [[G-24] Gas Optimization: Using Assembly for Efficient Validation of `msg.sender`](#g-24-gas-optimization-using-assembly-for-efficient-validation-of-msgsender)


## [G-01] Gas Optimization: Integration of approveAndCheckIfNative Logic into callBridge function

**Explanation of Issue:**
The gas optimization report addresses the integration of the `approveAndCheckIfNative` logic directly into the `callBridge` function. This modification aims to reduce redundancy and gas consumption by avoiding separate function calls and optimizing storage mapping accesses.

  - *Non-optimized Code:*
```solidity
    function callBridge(
        uint256 amt2Bridge,
        uint bridgeFee,
        BridgeInstructions memory instructions
    ) private returns (bytes memory) {
        bool native = approveAndCheckIfNative(instructions, amt2Bridge);
        return
            IBridgeAdapter(bridgeAdapters[instructions.bridgeId]).bridge{
                value: bridgeFee + (native ? amt2Bridge : 0)
            }(
                amt2Bridge,
                instructions.postBridge,
                instructions.dstChainId,
                instructions.target,
                instructions.paymentOperator,
                instructions.payload,
                instructions.additionalArgs,
                instructions.refund
            );
    }

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
https://github.com/code-423n4/2024-01-decent/blob/main//src/UTB.sol#L282-L301

https://github.com/code-423n4/2024-01-decent/blob/main//src/UTB.sol#L207-L220

  - *Optimized Code:*
```solidity
    function callBridge(
        uint256 amt2Bridge,
        uint bridgeFee,
        BridgeInstructions memory instructions
    ) private returns (bytes memory) {
        address adapter = bridgeAdapters[instructions.bridgeId];
        IBridgeAdapter bridgeAdapter = IBridgeAdapter(adapter);

        address bridgeToken = bridgeAdapter.getBridgeToken(instructions.additionalArgs);

        if (bridgeToken != address(0)) {
            IERC20(bridgeToken).approve(address(bridgeAdapter), amt2Bridge);
            return bridgeAdapter.bridge{
                value: bridgeFee + amt2Bridge
            }(
                amt2Bridge,
                instructions.postBridge,
                instructions.dstChainId,
                instructions.target,
                instructions.paymentOperator,
                instructions.payload,
                instructions.additionalArgs,
                instructions.refund
            );
        } else {
            return bridgeAdapter.bridge{
                value: bridgeFee
            }(
                amt2Bridge,
                instructions.postBridge,
                instructions.dstChainId,
                instructions.target,
                instructions.paymentOperator,
                instructions.payload,
                instructions.additionalArgs,
                instructions.refund
            );
        }
    }
```

**Reason:**  
The optimization involves integrating the logic of `approveAndCheckIfNative` directly into the `callBridge` function. The reasons for this optimization are:

1. **Redundancy Reduction:**
   Combining the logic eliminates the need for a separate function call (`approveAndCheckIfNative`), reducing redundancy and avoiding unnecessary overhead associated with additional function invocations.

2. **Storage Access Optimization:**
   The `adapter` variable is used to cache the result of `bridgeAdapters[instructions.bridgeId]` directly within the `callBridge` function, optimizing storage access by avoiding redundant mappings.

3. **Code Readability:**
   Consolidating the logic into a single function enhances code readability by providing a more straightforward and self-contained view of the bridge calling process. This contributes to better code maintainability.

By integrating the logic directly into the `callBridge` function, gas efficiency is improved, resulting in potential savings during contract execution. The modified code remains clear and concise while adhering to best practices in smart contract development.


## [G-02] Gas Optimization: Prefer Low-Level Calls Over Contract Existence Checks

**Explanation of Issue:**

In Solidity versions prior to 0.8.10, the compiler inserted additional code, including the `EXTCODESIZE` operation (costing 100 gas), to check for the existence of a contract before making an external function call. However, in more recent Solidity versions, these existence checks are skipped if the external call has a return value. To achieve similar gas-efficient behavior in earlier versions, low-level calls can be used since they inherently bypass contract existence checks.

**Examples of Code:**

  - *Non-optimized Code:*
    ```solidity
    contract NonOptimizedContract {
        function externalCallWithExistenceCheck(address _target) public view returns (uint256) {
            // Non-optimized: External call with compiler-inserted contract existence check
            return SomeExternalContract(_target).someFunction();
        }
    }
    ```

  - *Optimized Code:*
    ```solidity
    contract OptimizedContract {
        function externalCallWithoutExistenceCheck(address _target) public view returns (uint256) {
            // Optimized: External call using low-level call, bypassing existence check
            (bool success, bytes memory result) = _target.staticcall(abi.encodeWithSignature("someFunction()"));
            require(success, "External call failed");
            return abi.decode(result, (uint256));
        }
    }
    ```

**Reason:**  

The non-optimized code relies on the compiler-inserted contract existence check, which can incur an additional gas cost. In contrast, the optimized code uses a low-level call (`staticcall`) to directly invoke the external function without a contract existence check.
By utilizing low-level calls, the optimized code achieves the same behavior as the recent Solidity versions without existence checks, making it more gas-efficient. This optimization is particularly relevant in scenarios where external calls are made frequently and the contract existence is guaranteed.



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


## [G-03] Gas Optimization: Change Public State Variable Visibility to Private

**Explanation of Issue:**
The current state variables in the contract have public visibility, which exposes them to potential security risks, increases complexity, and incurs higher gas costs. It is recommended to change the visibility to private unless there is a specific need for public access.

**Examples of Code:**
  - *Non-optimized Code:*
    ```solidity
    contract ExampleContract {
        // Non-optimized: Public state variable
        uint256 public publicVariable;

        // Additional code...
    }
    ```
  - *Optimized Code:*
    ```solidity
    contract ExampleContract {
        // Optimized: Private state variable
        uint256 private privateVariable;

        // Additional code...
    }
    ```

**Reason:**  
1. **Security:** Public state variables are susceptible to unauthorized access and modification, posing security risks to the contract. Changing the visibility to private limits access to these variables, reducing the potential for malicious attacks.

2. **Encapsulation:** Private state variables encapsulate the internal workings of the contract, promoting better code organization and readability. By exposing only necessary interfaces, the contract becomes easier to understand and maintain.

3. **Gas Costs:** Public state variables come with higher gas costs for reading and writing, as Solidity generates additional getter and setter functions. Using private state variables helps reduce gas costs, making the contract more efficient in terms of execution on the Ethereum blockchain.

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


## [G-04] Gas Optimization: Avoiding address(this) in Favor of Hardcoded Addresses

**Explanation of Issue:**

In certain scenarios, opting for a hardcoded address over the use of `address(this)` can lead to more gas-efficient smart contracts. This recommendation is particularly relevant when the same contract address is required multiple times within the contract code.

**Examples of Code:**

  - *Non-optimized Code:*
    ```solidity
    contract MyContract {
        #L
        function doSomething() public {
            // Using address(this)
            require(msg.sender == address(this), "Caller is not authorized");
            
            // Do something
        }
    }
    ```

  - *Optimized Code:*
    ```solidity
    contract MyContract {
        address public immutable myAddress = 0x1234567890123456789012345678901234567890;
        #L
        function doSomething() public {
            // Using hardcoded address
            require(msg.sender == myAddress, "Caller is not authorized");
            
            // Do something
        }
    }
    ```

**Reason:**  

The optimization stems from the fact that utilizing `address(this)` necessitates an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode. This additional operation contributes to increased gas costs. By pre-calculating and incorporating a hardcoded address, the need for the extra operation is eliminated, resulting in a reduction in the overall gas cost of the contract.

This optimization becomes particularly impactful when the contract address is referenced multiple times throughout the code. In such cases, the use of a hardcoded address can significantly enhance the gas efficiency of the smart contract.
[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)


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

## [G-05] Gas Optimization: Prefer Storage over Memory for Structs/Arrays

**Explanation of Issue:**

When dealing with structs or arrays and fetching data from a storage location, opting for memory variables can result in higher gas costs. This is because assigning the data to a memory variable causes all fields of the struct or array to be read from storage, incurring a Gcoldsload (2100 gas) for each field.

Instead of declaring the variable with the memory keyword, it is more gas-efficient to declare it with the storage keyword. Additionally, caching any fields that need to be re-read in stack variables can further optimize gas usage. This approach ensures that only the fields actually read incur the Gcoldsload, resulting in substantial gas savings.

**Examples of Code:**

  - *Non-optimized Code:*
    ```solidity
    contract MyContract {
        struct MyStruct {
            uint256 field1;
            uint256 field2;
            // ... more fields
        }
        
        function fetchData() public view returns (uint256) {
            MyStruct memory myData = storageData; // Gas-inefficient
            return myData.field1;
        }
        
        MyStruct storage storageData; // Assume storageData is assigned elsewhere
    }
    ```

  - *Optimized Code:*
    ```solidity
    contract MyContract {
        struct MyStruct {
            uint256 field1;
            uint256 field2;
            // ... more fields
        }
        
        function fetchData() public view returns (uint256) {
            MyStruct storage myData = storageData; // Gas-optimized
            return myData.field1;
        }
        
        MyStruct storage storageData; // Assume storageData is assigned elsewhere
    }
    ```

**Reason:**  

The optimization lies in minimizing gas costs associated with storage reads. When declaring variables with the memory keyword, the gas cost increases due to Gcoldsload operations for each field of the struct or array. By utilizing the storage keyword and selectively caching fields in stack variables, unnecessary Gcoldsload operations are avoided, leading to more gas-efficient smart contracts.
Reading the whole struct or array into a memory variable makes sense only when the 



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

## [G-06] bytes constants are more efficient than string constants

**Explanation of Issue:** If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less expensive in terms of gas consumption.

**Examples of Code:**
- *Non-optimized Code:*
```solidity
string public constant myString = "Hello, world!";
```

- *Optimized Code:*
```solidity
bytes32 public constant myBytes = bytes32("Hello, world!");
```

**Reason:**

Using bytes32 instead of strings for constants that fit within 32 bytes can lead to gas savings. Here's why:

1. **Gas Cost:** The storage and memory costs of working with bytes32 are lower compared to strings. Strings are dynamic arrays in Solidity, requiring additional gas for storage and memory allocation. Using bytes32 for small constant values eliminates these extra costs.

2. **Efficiency:** bytes32 is a fixed-size data type, while strings are variable-length. By using bytes32, you ensure that the storage and memory requirements are constant, resulting in more efficient contract execution.

As a smart contract Solidity auditor and gas optimizer, it is important to consider the size and nature of the data when choosing between bytes32, bytes, and strings. Opting for bytes32 instead of strings for small constant values can lead to gas savings without compromising functionality. However, it's crucial to verify that the data can indeed fit within 32 bytes to avoid truncation or unexpected behavior.

There are 1 instances of this issue:
```solidity
File: src/UTBFeeCollector.sol
10   string constant BANNER = "\x19Ethereum Signed Message:\n32";
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L10

## [G-07] Gas Optimization: Use Assembly for Efficient Hashing

**Explanation of Issue:**

When hashing values, employing assembly instead of Solidity can lead to significant gas savings. This optimization is particularly evident when using the keccak256 function.

**Examples of Code:**

  - *Non-optimized Code:*
    ```solidity
    contract NonOptimizedContract {
        function solidityHash(uint256 a, uint256 b) public view returns (bytes32) {
            // Unoptimized
            return keccak256(abi.encodePacked(a, b));
        }
    }
    ```

  - *Optimized Code:*
    ```solidity
    contract OptimizedContract {
        function assemblyHash(uint256 a, uint256 b) public view returns (bytes32) {
            // Optimized
            bytes32 hashedVal;
            assembly {
                mstore(0x00, a)
                mstore(0x20, b)
                hashedVal := keccak256(0x00, 0x40)
            }
            return hashedVal;
        }
    }
    ```

**Reason:**  

The optimization involves using assembly to directly hash values, resulting in substantial gas savings. In the non-optimized code, the Solidity approach using `keccak256(abi.encodePacked(a, b))` incurs a higher gas cost.
In the optimized code, the assembly approach utilizes the `mstore` instruction to efficiently load values into memory and then applies the keccak256 function. This results in a more gas-efficient hashing operation, as demonstrated by the reduced gas cost in the optimized code.
Utilizing assembly for hashing is particularly advantageous when optimizing for gas costs in smart contracts, making it a valuable technique for gas-conscious developers.

There are 1 instances of this issue:
```solidity
File: src/UTBFeeCollector.sol
49      bytes32 constructedHash = keccak256(
            abi.encodePacked(BANNER, keccak256(packedInfo))
        );
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L49-L51

## [G‑08] Gas Optimization Issue - Redundant Stack Variable Assignment

**Explanation of Issue:**
When a stack variable is employed as a transient cache for a state variable, and this stack variable is only accessed once, it leads to unnecessary gas consumption. In such cases, it is more gas-efficient to use the state variable directly, avoiding the additional gas cost associated with the redundant stack assignment.

**Examples of Code:**
*Non-optimized Code:*
```solidity
uint256 public totalBalance;

function updateBalance(uint256 newAmount) external {
    uint256 tempBalance = totalBalance; // Redundant stack assignment
    // Operations on tempBalance
    totalBalance = tempBalance + newAmount;
}
```

*Optimized Code:*
```solidity
uint256 public totalBalance;

function updateBalance(uint256 newAmount) external {
    // Operations directly on totalBalance
    totalBalance = totalBalance + newAmount; // Gas-efficient, eliminates redundant stack assignment
}
```

**Reason:**
In the non-optimized example, a stack variable (`tempBalance`) is introduced to temporarily hold the value of the state variable (`totalBalance`). However, since `tempBalance` is only accessed once, it is more gas-efficient to perform the operations directly on the state variable, eliminating the need for an unnecessary stack assignment. This optimization reduces gas consumption, contributing to a more economical contract execution.

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


## [G-09]Gas Optimization Issue - Gas-Efficient Alternatives for Common Math Operations

**Explanation of Issue:**
Common math operations, such as min and max, may have more gas-efficient alternatives. In scenarios where unoptimized code uses conditional operators, like the ternary operator, the resulting conditional jumps in opcodes can incur higher gas costs. Exploring gas-efficient alternatives for these operations is crucial for optimizing smart contract performance.

**Examples of Code:**
*Non-optimized Code:*
```solidity
function max(uint256 x, uint256 y) public pure returns (uint256 z) {
    z = x > y ? x : y; // Unoptimized conditional operator
}
```

*Optimized Code:*
```solidity
function max(uint256 x, uint256 y) public pure returns (uint256 z) {
    /// @solidity memory-safe-assembly
    assembly {
        z := xor(x, mul(xor(x, y), gt(y, x))) // Gas-efficient alternative
    }
}
```

**Reason:**
In the non-optimized example, the ternary operator is used for the max function, introducing conditional jumps in the opcodes, which can be gas-costly. The optimized code provides a gas-efficient alternative using assembly code. By avoiding conditional operators, the gas consumption is reduced, leading to improved contract efficiency. The Solady Library offers additional gas-efficient math operations, making it valuable for developers seeking optimization opportunities.

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




## [G-10] Using a positive conditional flow to save a NOT opcode

**Explanation of Issue:** By structuring conditional flows in a positive manner, you can save gas by avoiding the use of the NOT opcode.

**Examples of Code:**
- *Non-optimized Code:*
```solidity
bool public isTrue;

function checkCondition() public {
    if (!isTrue) {
        // Code to be executed if condition is false
    } else {
        // Code to be executed if condition is true
    }
}
```

- *Optimized Code:*
```solidity
bool public isTrue;

function checkCondition() public {
    if (isTrue) {
        // Code to be executed if condition is true
    } else {
        // Code to be executed if condition is false
    }
}
```

**Reason:**

The optimization revolves around avoiding the use of the NOT opcode (!) in the conditional flow. Here's the reason behind this optimization:

1. **Gas Cost:** Using a positive conditional flow eliminates the need for the NOT opcode, resulting in gas savings. The NOT opcode consumes gas when evaluating the negation of a boolean expression. By structuring the conditional flow in a positive manner, the NOT opcode is avoided, leading to reduced gas consumption.
As a smart contract Solidity auditor and gas optimizer, it is important to consider these minor optimizations that can collectively contribute to reducing gas costs. However, it's crucial to ensure that the optimized code maintains its intended functionality and readability. Gas optimization should be done carefully, considering the trade-offs between gas cost and code maintainability.

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


## [G-11] Gas Optimization: Using Assembly for Efficient Back-to-Back External Calls

**Explanation of Issue:**

Performing consecutive external calls with similar function signatures can result in redundant operations and increased gas costs. The use of assembly provides an opportunity to optimize gas consumption by reusing function signatures, parameters, and memory space.

**Examples of Code:**
  - *Non-optimized Code:*
    ```solidity
    contract NonOptimizedContractA {
        function nonOptimizedCalls(uint256 param) public {
            // Non-optimized back-to-back calls
            ExternalContractB.externalFunctionB(param);
            ExternalContractC.externalFunctionC(param);
        }
    }

    contract ExternalContractB {
        function externalFunctionB(uint256 param) external {
            // Perform external call B
            // ...
        }
    }

    contract ExternalContractC {
        function externalFunctionC(uint256 param) external {
            // Perform external call C
            // ...
        }
    }
    ```

  - *Optimized Code:*
    ```solidity
    contract OptimizedContractD {
        address public externalContractBAddress;
        address public externalContractCAddress;

        constructor(address _externalContractBAddress, address _externalContractCAddress) {
            externalContractBAddress = _externalContractBAddress;
            externalContractCAddress = _externalContractCAddress;
        }

        function optimizedCalls(uint256 param) public {
            // Optimized back-to-back calls using assembly
            assembly {
                // Cache the free memory pointer
                let freeMemoryPointer := mload(0x40)

                // Perform external call to ExternalContractB.externalFunctionB
                let successB := call(gas(), sload(externalContractBAddress.slot), 0, 0, param, 0, 0)

                // Restore the free memory pointer
                mstore(0x40, freeMemoryPointer)

                // Reset the zero slot if necessary
                if eq(successB, 0) {
                    sstore(0, 0)
                }

                // Cache the free memory pointer again
                freeMemoryPointer := mload(0x40)

                // Perform external call to ExternalContractC.externalFunctionC
                let successC := call(gas(), sload(externalContractCAddress.slot), 0, 0, param, 0, 0)

                // Restore the free memory pointer
                mstore(0x40, freeMemoryPointer)

                // Reset the zero slot if necessary
                if eq(successC, 0) {
                    sstore(0, 0)
                }
            }
        }
    }
    ```

**Reason:**  

The non-optimized code involves separate external calls within `NonOptimizedContractA`, resulting in redundant operations and potential memory expansion costs.
In the optimized code (`OptimizedContractD`), assembly is used to efficiently reuse function signatures, parameters, and memory space for back-to-back external calls to `ExternalContractB.externalFunctionB` and `ExternalContractC.externalFunctionC`. The free memory pointer is cached and restored, reducing potential memory expansion costs. This optimization results in a more gas-efficient implementation when similar external calls are performed consecutively.


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


## [G-12] Title: Gas Optimization Issue in External/Internal Function Calls

**Explanation of Issue:**
The issue lies in the non-optimized code of ContractB, where unnecessary external calls are made to functions of ContractA. This results in increased gas consumption, as the same external calls are redundantly invoked within the internal functions.

**Examples of Code:**

*Non-optimized Code:*
```solidity
// Contract B
contract ContractB {
    // External reference to ContractA

    function AB(uint256 _valueR, uint256 _valueT,address _token) public {
        contractA(_token).R(_valueR);

        // External call to T function of ContractA
        contractA(_token).T(_valueT);
    }
}
```

*Optimized Code:*
```solidity
// Contract B
contract ContractB {
    // External reference to ContractA

    function AB(uint256 _valueR, uint256 _valueT,address _token) public {
        ContractA  contractA = ContractA(_token);
        contractA.R(_valueR);

        // External call to T function of ContractA
        contractA.T(_valueT);
    }
}
```

**Reason:**
In the non-optimized code, unnecessary external calls to ContractA's functions are made separately in the `AB` function of ContractB. The optimized code combines these external calls, avoiding redundant calls and improving gas efficiency. By caching the reference to ContractA, we eliminate redundant external function invocations, resulting in reduced gas costs during contract execution.

There are 7 instance(s) of this issue:

- First, cache the result of `IERC20(swapParams.tokenIn)` in an `IERC20` type variable, and then call the `transferFrom` and `approve` functions in side `UTB.performSwap` function.
```solidity
File: src/UTB.sol
83  IERC20(swapParams.tokenIn).transferFrom(

90  IERC20(swapParams.tokenIn).approve(    
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L83


- First, cache the result of `IERC20(fees.feeToken)` in an `IERC20` type variable, and then call the `transferFrom` and `approve` functions inside `UTB.retrieveAndCollectFees` modifier.
```solidity
File: src/UTB.sol
                IERC20(fees.feeToken).transferFrom(
                    msg.sender,
                    address(this),
                    fees.feeAmount
                );
                IERC20(fees.feeToken).approve(
                    address(feeCollector),
                    fees.feeAmount
                );
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L236-L244


- First, cache the result of `IERC20(token)` in an `IERC20` type variable, and then call the `transferFrom` and `approve` and  `transfer` functions inside `UTBExecutor.execute` function.
```solidity
File: src/UTBExecutor.sol
61        IERC20(token).transferFrom(msg.sender, address(this), amount);
62        IERC20(token).approve(paymentOperator, amount);
80        IERC20(token).transfer(refund, remainingBalance);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L61-L62


- First, cache the result of `IERC20(bridgeToken)` in an `IERC20` type variable, and then call the `transferFrom` and `approve` functions inside `DecentBridgeAdapter.bridge` function.
```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol
109         IERC20(bridgeToken).transferFrom(
                msg.sender,
                address(this),
                amt2Bridge
            );
            IERC20(bridgeToken).approve(address(router), amt2Bridge);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L109-L114


- First, cache the result of `IERC20(swapParams.tokenIn)` in an `IERC20` type variable, and then call the `transferFrom` and `approve` functions inside `DecentBridgeAdapter.receiveFromBridge` function.
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


- First, cache the result of `IERC20(bridgeToken)` in an `IERC20` type variable, and then call the `transferFrom` and `approve` functions inside `StargateBridgeAdapter.bridge` function.
```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol
88        IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);
89        IERC20(bridgeToken).approve(address(router), amt2Bridge);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88-L89

## [G-13] Gas Optimization: Favor Ternary Operation Over If-Else Statement

**Explanation of Issue:**

Using a ternary operation instead of an if-else statement can result in reduced gas costs, making it a more gas-efficient choice for conditional assignments.

**Examples of Code:**

  - *Non-optimized Code:*
    ```solidity
    contract NonOptimizedContract {
        function conditionallyAssign(uint256 value) public pure returns (uint256) {
            // Non-optimized: Using if-else statement for conditional assignment
            if (value > 0) {
                return value;
            } else {
                return 0;
            }
        }
    }
    ```

  - *Optimized Code:*
    ```solidity
    contract OptimizedContract {
        function conditionallyAssign(uint256 value) public pure returns (uint256) {
            // Optimized: Using ternary operation for conditional assignment
            return (value > 0) ? value : 0;
        }
    }
    ```

**Reason:**  

The non-optimized code utilizes an if-else statement for a simple conditional assignment. While this approach is readable, it involves additional opcodes, potentially leading to higher gas costs.
In the optimized code, a ternary operation is used for the same conditional assignment. Ternary operations generally result in fewer opcodes and are therefore more gas-efficient. This optimization is particularly valuable in scenarios where gas-conscious development is a priority, and the conditionals involve simple assignments.

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



## [G-14] Unlimited gas consumption risk due to external call recipients

**Explanation of Issue:** When calling an external function without specifying a gas limit, the called contract may consume all the remaining gas, causing the transaction to be reverted. To mitigate this risk, it is recommended to explicitly set a gas limit when making low-level external calls.

**Examples of Code:**

- *Non-optimized Code:*
```solidity
contract A {
    function callExternalContract(address externalContract, uint256 amount, bytes memory payload) external {
        (bool success, ) = externalContract.call{value: amount}(payload);
        require(success, "External call failed");
    }
}
```

- *Optimized Code:*
```solidity
contract A {
    function callExternalContract(address externalContract, uint256 amount, bytes memory payload) external {
        (bool success, ) = externalContract.call{value: amount, gas: 200000}(payload);
        require(success, "External call failed");
    }
}
```

**Reason:**

The optimization involves explicitly setting a gas limit when making low-level external calls to mitigate the risk of unlimited gas consumption. Here's the reasoning behind this optimization, considering the provided code snippet:

1. **Gas Consumption Risk:** When an external function is called without specifying a gas limit, the called contract has the potential to consume all the remaining gas. This can cause the transaction to be reverted and result in unexpected behavior.

2. **Gas Limit Mitigation:** By explicitly setting a gas limit when making low-level external calls, such as `externalContract.call{value: amount, gas: 200000}(payload)`, you can control the maximum amount of gas that can be consumed by the called contract. This helps mitigate the risk of unlimited gas consumption and ensures that the transaction remains within the desired gas boundaries.

3. **Value and Payload:** The provided code also demonstrates passing an amount of Ether (`value`) and a data payload (`payload`) to the external contract. These considerations are essential when making external calls, but the gas optimization mainly focuses on mitigating the gas consumption risk.

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

## [G-15] Gas Optimization: Prefer `bytes.concat()` Over `abi.encodePacked` for Non-Hashing Concatenation

**Explanation of Issue:**

When concatenating bytes and not intending to use the result for hashing, `bytes.concat()` is a more gas-efficient alternative to `abi.encodePacked`. Using `bytes.concat()` can lead to reduced gas costs in scenarios where concatenation is performed on bytes variables.

**Examples of Code:**

  - *Non-optimized Code:*
    ```solidity
    contract NonOptimizedContract {
        function concatenateBytes(bytes memory a, bytes memory b) public pure returns (bytes memory) {
            // Non-optimized: Using abi.encodePacked for concatenation
            return abi.encodePacked(a, b);
        }
    }
    ```

  - *Optimized Code:*
    ```solidity
    contract OptimizedContract {
        function concatenateBytes(bytes memory a, bytes memory b) public pure returns (bytes memory) {
            // Optimized: Using bytes.concat for concatenation
            return bytes.concat(a, b);
        }
    }
    ```

**Reason:**  

The non-optimized code uses `abi.encodePacked` for concatenating bytes. While this approach is valid, it may result in higher gas costs, especially when concatenating large amounts of data.
In the optimized code, `bytes.concat()` is utilized for non-hashing concatenation. This method is specifically designed for concatenating bytes variables and is generally more gas-efficient than `abi.encodePacked`. By adopting this optimization, gas consumption can be reduced, making the smart contract more cost-effective in scenarios where byte concatenation is a common operation.


There are 1 instances of this issue:
```solidity
File: src/UTBFeeCollector.sol
        bytes32 constructedHash = keccak256(
            abi.encodePacked(BANNER, keccak256(packedInfo))
        );
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L49-L51






## [G-16] Optimize External Calls with Assembly for Memory Efficiency

**Explanation of Issue:** Using interfaces to make external contract calls in Solidity can be inefficient in terms of memory utilization. Each such call involves creating a new memory location to store the data being passed, resulting in memory expansion costs. This can impact gas consumption, especially for contracts that make frequent external calls. However, using inline assembly allows for optimized memory usage by reusing already allocated memory spaces or utilizing scratch space for smaller datasets. This optimization can lead to notable gas savings. Furthermore, inline assembly enables important safety checks, such as verifying if the target address has code deployed to it using `extcodesize(addr)` before making the call, mitigating risks associated with contract interactions.

**Examples of Code:**

- *Non-optimized Code:*
```solidity
interface ExternalContract {
    function someFunction(uint256 data) external returns (uint256);
}

contract MyContract {
    ExternalContract externalContract;

    function callExternalContract(uint256 data) external returns (uint256) {
        return externalContract.someFunction(data);
    }
}
```

- *Optimized Code:*
```solidity
contract MyContract {
    function callExternalContract(address externalContract, uint256 data) external returns (uint256) {
        uint256 result;
        bool success;
        
        assembly {
            let freeMemoryPointer := mload(0x40) // Get the current free memory pointer
            
            // Verify if the target address has code deployed to it
            let codeSize := extcodesize(externalContract)
            if iszero(codeSize) {
                revert(0, 0)
            }
            
            // Prepare the data to be passed to the external contract
            mstore(freeMemoryPointer, data)
            let inputData := freeMemoryPointer
            let inputSize := 32
            
            // Make the external call
            success := call(gas(), externalContract, 0, inputData, inputSize, freeMemoryPointer, 32)
            
            // Retrieve the result from the external call
            result := mload(freeMemoryPointer)
        }
        
        require(success, "External call failed");
        return result;
    }
}
```

**Reason:**

The optimization involves using inline assembly to optimize memory usage and improve gas efficiency when making external contract calls. Here's the reasoning behind this optimization:

1. **Memory Efficiency:** Using interfaces for external calls in Solidity incurs memory expansion costs, as each call involves creating a new memory location to store the data being passed. By utilizing inline assembly, memory can be allocated and reused more efficiently, resulting in reduced gas consumption.

2. **Gas Savings:** Optimized memory usage through inline assembly can lead to significant gas savings, especially for contracts that make frequent external calls. Reusing already allocated memory spaces or utilizing scratch space for smaller datasets avoids unnecessary memory expansion costs.

3. **Safety Checks:** Inline assembly allows for important safety checks before making external calls. For example, using `extcodesize(addr)` verifies if the target address has code deployed to it, mitigating risks associated with calling non-existent contracts.


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


## [G-17] Sort Solidity operations using short-circuit mode

**Explanation of Issue:**
The issue involves optimizing the sequencing of Solidity operations using short-circuit mode. Short-circuiting utilizes OR/AND logic to arrange operations based on their gas costs. By placing low gas cost operations at the front and high gas cost operations at the back, subsequent high-cost Ethereum virtual machine operations can be skipped (short-circuited) if the initial low-cost operation evaluates to true.

**Examples of Code:**
- f(x) is a low gas cost operation 
- g(y) is a high gas cost operation 
- *Non-optimized Code:*
```solidity
// Operations not sorted by gas cost
function performOperations(uint256 x, uint256 y) public {
    // High gas cost operation
    if (g(y) || f(x)) {
        // Code to execute if the condition is true
    }
}
```

- *Optimized Code:*
```solidity
// Operations sorted by gas cost using short-circuit mode
function performOperations(uint256 x, uint256 y) public {
    // Low gas cost operation first
    if (f(x) || g(y)) {
        // Code to execute if the condition is true
    }
}
```

**Reason:**
The reason for this optimization is to improve gas efficiency by leveraging short-circuiting. By sorting operations based on their gas costs, with low-cost operations placed before high-cost operations, subsequent high-cost Ethereum virtual machine operations can be skipped if the preceding low-cost operation evaluates to true. In the non-optimized code example, the `g(y)` operation is executed first, and if it returns true (or a non-zero value), the `f(x)` operation is not executed due to the short-circuit behavior of the `||` operator. However, this does not follow the intended gas optimization technique. In the optimized code example, the `f(x)` operation is executed first, and if it returns true, the `g(y)` operation is not executed. This optimization reduces unnecessary gas consumption and results in cost savings during contract execution.


There are 2 instances of this issue:

- Accessing the state variable `gasCurrencyIsEth` consumes more gas compared to accessing the function parameter `deliverEth`.
```solidity
File: src/DecentBridgeExecutor.sol
77   if (!gasCurrencyIsEth || !deliverEth) {
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L77

fix code :

```diff
-   if (!gasCurrencyIsEth || !deliverEth) {
+   if (!deliverEth || !gasCurrencyIsEth) {
```

```solidity
File: src/DecentEthRouter.sol
272   if (!gasCurrencyIsEth || !deliverEth) {
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L272



## [G-18] Gas Optimization: Use Assembly To Check For address(0)
#### `note` : All instances are not found by bot race
**Explanation of Issue:**
Gas optimization is a critical aspect of smart contract development, and identifying areas for improvement is essential. In this case, it is observed that using assembly to check for address(0) can result in significant gas savings.

**Examples of Code:**
  - *Non-optimized Code:*
    ```solidity
    // Solidity code without assembly optimization
    if (_addr == address(0)) {
        revert("zero address");
    }
    ```
  - *Optimized Code:*
    ```solidity
    // Solidity code with assembly optimization
    assembly {
        if iszero(_addr) {
            mstore(0x00, "zero address");
            revert(0x00, 0x20);
        }
    }
    ```

**Reason:**
The gas optimization is achieved by using assembly to explicitly check for the zero address condition. In the non-optimized code, the equality check is performed in high-level Solidity, which may result in higher gas consumption. By leveraging assembly, a direct comparison with the zero address can be made, and if the condition is met, gas-efficient operations such as `mstore` and `revert` are utilized.


There are 2 instances of this issue:

```solidity
File:  src/UTBFeeCollector.sol
55  if (fees.feeToken != address(0)) {

70  if (token == address(0)) {
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L55


## [G-19] Gas Optimization: Use Assembly in Place of `abi.decode` to Extract Calldata Values More Efficiently
#### `note` : All instances are not found by bot race
**Explanation of Issue:**
The current gas optimization report highlights the use of assembly over `abi.decode` for extracting calldata values. By leveraging assembly, we can enhance the efficiency of decoding calldata values, ensuring that only the necessary values are decoded. This optimization is particularly beneficial in scenarios where there is a need to minimize gas consumption during contract execution.

**Examples of Code:**
  - *Non-optimized Code:*
    ```solidity
    // Solidity code using abi.decode for calldata decoding
    function extractData() external {
        (uint256 value1, uint256 value2) = abi.decode(msg.data, (uint256, uint256));
        // Further code logic using value1 and value2
    }
    ```
  - *Optimized Code:*
    ```solidity
    // Solidity code using assembly for efficient calldata decoding
    function extractData() external {
        uint256 value1;
        uint256 value2;
        
        assembly {
            // Offset of calldata values to be extracted (assuming 4-byte values)
            let offset := add(msg.data, 4)
            
            // Load values directly from calldata
            value1 := mload(offset)
            value2 := mload(add(offset, 32))
        }
        
        // Further code logic using value1 and value2
    }
    ```

**Reason:**  
The optimization revolves around using assembly to directly extract calldata values instead of relying on the higher-level `abi.decode` function. The non-optimized code utilizes `abi.decode` to extract values, potentially decoding more information than needed. In contrast, the optimized code in assembly allows for precise extraction, loading only the necessary values directly from calldata.


There are 8 instances of this issue:
```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol
51  SwapParams memory swapParams = abi.decode(

96  uint64 dstGas = abi.decode(additionalArgs, (uint64));

103 SwapParams memory swapParams = abi.decode(

134 SwapParams memory swapParams = abi.decode(    
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L51


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol
202   SwapParams memory swapParams = abi.decode(
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L202


```solidity
File: src/DecentEthRouter.sol
251  (, , , , callPayload) = abi.decode(
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L251


```solidity
File: src/UTB.sol
69   SwapParams memory swapParams = abi.decode(

181  SwapParams memory newPostSwapParams = abi.decode(    
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L69


## [G-20] Calldata should be used in place of memory function parameters when not mutated
#### `note` : All instances are not found by bot race
**Explanation of Issue:**
In Solidity, when an external function does not modify function parameters of type `bytes memory`, `string memory`, or `[] memory`, it is recommended to use `calldata` instead of `memory` for these parameters. `calldata` is a non-modifiable, non-persistent area where function arguments are stored, and it is generally cheaper in terms of gas compared to `memory`. This optimization is particularly beneficial for arrays and complex data types when used in external function calls. 

**Examples of Code:**
- *Non-optimized Code:*
```solidity
function processArray(uint256[] memory data) external {
    // Function logic using the data array
}
```

- *Optimized Code:*
```solidity
function processArray(uint256[] calldata data) external {
    // Function logic using the data array
}
```

**Reason:**  
Using `calldata` instead of memory for function parameters in external functions saves gas by avoiding the need for copying data to `memory`. This approach provides direct read-only access to function arguments from the input data, resulting in reduced gas consumption. It is particularly beneficial for large arrays or complex data types, as it eliminates the gas cost associated with copying data to memory. This optimization improves contract efficiency, reduces costs for users, and enhances overall performance.

There are 5 instance(s) of this issue:
```solidity
File: src/DecentEthRouter.sol
243   bytes memory _payload
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L243


```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol
185   bytes memory, // _srcAddress

189   bytes memory payload
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L185


```solidity
File: src/swappers/UniSwapper.sol
33  SwapParams memory newSwapParams,

59  bytes memory swapPayload
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L33





## [G-21] Use custom errors rather than revert()/require() strings to save gas
#### `note` : All instances are not found by bot race
**Explanation of Issue:**
Using custom errors instead of revert()/require() strings can save gas. Custom errors are available from Solidity version 0.8.4.

**Examples of Code:**
  - *Non-optimized Code:*
    ```
    function foo() public {
        require(msg.sender == owner, "Only the owner can call this function.");
        // ...
    }
    ```
  - *Optimized Code:*
    ```
    error OnlyOwner();
    function foo() public {
        if (msg.sender != owner) {
            revert OnlyOwner();
        }
        // ...
    }
    ```

**Reason:**
Custom errors are more efficient than revert()/require() strings because they consume less gas. They also provide more flexibility in error handling and can be used to provide more detailed error messages to users.

There are 2 instance(s) of this issue:
```solidity
File:  src/UTBFeeCollector.sol
29    require(signature.length == 65, "Invalid signature length");

54    require(recovered == signer, "Wrong signature");
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L29


## [G-22] Gas Optimization: Marking Constructors as Payable
#### `note` : All instances are not found by bot race
**Explanation of Issue:**
The gas optimization report highlights the opportunity to mark constructors as payable in Solidity contracts. Payable functions, including constructors, incur lower gas costs compared to non-payable functions. Marking constructors as payable can result in gas savings, as the compiler eliminates extra checks that ensure payments are not provided. In the case of constructors, this optimization is safe because only the deployer has the ability to pass funds during contract deployment.

**Examples of Code:**
  - *Non-optimized Code:*
    ```solidity
    contract MyContract {
        uint256 public value;

        // Non-payable constructor
        constructor(uint256 _initialValue) {
            value = _initialValue;
        }
    }
    ```
  - *Optimized Code:*
    ```solidity
    contract MyContract {
        uint256 public value;

        // Payable constructor
        constructor(uint256 _initialValue) payable {
            value = _initialValue;
        }
    }
    ```

**Reason:**  
The optimization involves marking the constructor as payable. The reasons for this optimization are:

1. **Gas Cost Reduction:**
   Payable functions, including constructors, have lower gas costs because the compiler does not need to add extra checks to ensure that a payment wasn't provided. By marking the constructor as payable, unnecessary gas-consuming checks are eliminated.

2. **Deployer Exclusivity:**
   Constructors are executed only during contract deployment, and only the deployer has the ability to pass funds. As a result, marking the constructor as payable is safe, as it aligns with the nature of constructor execution.

3. **Avoidance of Extra Checks:**
   By explicitly marking the constructor as payable, the developer signals to the compiler that no additional checks for non-payability are required. This leads to cleaner and more efficient contract code.

This optimization is particularly relevant in scenarios where gas efficiency is a critical consideration, contributing to a more cost-effective deployment and initialization of smart contracts on the blockchain.

There are 2 instance(s) of this issue:
```solidity
File: src/DcntEth.sol
13    constructor(
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol

```solidity
File: src/UTBFeeCollector.sol
12  constructor() UTBOwned() {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L12



## [G-23] Avoid updating storage when the value hasn't changed to save gas 
#### `note` : All instances are not found by bot race
**Explanation of Issue:**
The issue is related to unnecessary storage updates when the old value is equal to the new value. This results in unnecessary gas consumption.

**Examples of Code:**
- *Non-optimized Code:*
```solidity
// Storage update without checking for value change
function updateValue(uint256 newValue) public {
    value = newValue;
}
```

- *Optimized Code:*
```solidity
// Avoiding storage update when the value hasn't changed
function updateValue(uint256 newValue) public {
    if (value != newValue) {
        value = newValue;
    }
}
```

**Reason:**
The reason for this optimization is to save gas by avoiding unnecessary storage updates. If the old value is equal to the new value, there is no need to update the storage, which saves the gas cost of a Gsreset operation (2900 gas). However, it's important to note that avoiding the storage update may result in a Gcoldsload operation (2100 gas) or a Gwarmaccess operation (100 gas) depending on the context of the contract execution.

There are 2 instance(s) of this issue:
```solidity
File: src/DcntEth.sol
21    router = _router;
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L21


```solidity
File: src/UTBFeeCollector.sol
19  signer = _signer;
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L19


## [G-24] Gas Optimization: Using Assembly for Efficient Validation of `msg.sender`
#### `note` : All instances are not found by bot race
**Explanation of Issue:**

When validating `msg.sender`, using assembly can significantly reduce the number of opcodes required, leading to a more gas-efficient implementation.

**Examples of Code:**

  - *Non-optimized Code:*
    ```solidity
    contract NonOptimizedContract {
        function validateSender() public view {
            // Non-optimized: Standard Solidity validation of msg.sender
            require(msg.sender == address(0x123...), "Unauthorized");
        }
    }
    ```

  - *Optimized Code:*
    ```solidity
    contract OptimizedContract {
        function validateSender() public view {
            // Optimized: Using assembly for efficient validation of msg.sender
            assembly {
                // Compare msg.sender directly in assembly
                if iszero(eq(sload(msg.sender.slot), 0x123...)) {
                    // Revert if the comparison fails
                    revert(0, 0)
                }
            }
        }
    }
    ```

**Reason:**  

The non-optimized code performs a standard Solidity validation of `msg.sender`, which involves multiple opcodes and can contribute to higher gas costs.
In the optimized code, assembly is used to efficiently validate `msg.sender` with fewer opcodes. The `iszero` and `eq` assembly functions directly compare the `msg.sender` value, resulting in a more gas-efficient implementation. This optimization is valuable when frequent and efficient validation of the sender's address is required in smart contracts.




There are 1 instance(s) of this issue:
```solidity
File: src/DcntEth.sol
9  require(msg.sender == router);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L9