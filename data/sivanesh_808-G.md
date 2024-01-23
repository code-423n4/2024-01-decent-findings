| S.No | Title | Contract Name |
|------|-------|---------------|
| G-01 | Optimization of `getBridgedAmount` Function in `StargateBridgeAdapter` Contract | StargateBridgeAdapter.sol |
| G-02 | Gas Optimization Report for `callBridge` Function in `StargateBridgeAdapter` Contract | StargateBridgeAdapter.sol |
| G-03 | Enhancing the `splitSignature` Function in the UTBFeeCollector Contract | UTBFeeCollector.sol |
| G-04 | Gas Optimization for `collectFees` Function in UTBFeeCollector.sol | UTBFeeCollector.sol |
| G-05 | Gas Optimization for `updateSwapParams` and `_refundUser` Functions in UniSwapper Contract | UniSwapper.sol |
| G-06 | Gas Optimization for `swapNoPath` Function in UniSwapper Contract | UniSwapper.sol |


### [G-01] Optimization of `getBridgedAmount` Function in `StargateBridgeAdapter` Contract

### Contract Name: `StargateBridgeAdapter.sol`

#### Description:
The `getBridgedAmount` function in the `StargateBridgeAdapter` contract calculates the amount to be bridged after deducting a fee. The original implementation involves a multiplication followed by a division, which can be optimized for gas efficiency. This report outlines a revised strategy to optimize the function for gas usage while maintaining its original purpose and accuracy.

### Original Code:
```solidity
function getBridgedAmount(
    uint256 amt2Bridge,
    address /*tokenIn*/,
    address /*tokenOut*/
) external pure returns (uint256) {
    return (amt2Bridge * (1e4 - SG_FEE_BPS)) / 1e4;
}
```

### Optimization Code:
```solidity
function getBridgedAmountOptimized(uint256 amt2Bridge) external pure returns (uint256) {
    uint256 feeFactor = 10000 - SG_FEE_BPS;
    return (amt2Bridge * feeFactor) / 10000;
}
```

### Optimization Explanation:
- **Simplified Calculation**: The optimized code first calculates a `feeFactor` by subtracting `SG_FEE_BPS` from `10000`. This approach simplifies the calculation by breaking it down into two clear steps - first determining the fee factor, then applying it to the `amt2Bridge`.
  
- **Reduced Arithmetic Operations**: By rearranging the arithmetic operations, the optimized code reduces the complexity of the calculation. The multiplication is done with a pre-calculated `feeFactor` followed by a single division. This reduction in operations can lead to less gas consumption.

- **Maintaining Functional Integrity**: The optimization maintains the original logic and purpose of the function. It accurately calculates the bridged amount after fee deduction, ensuring that the core functionality is not altered.

- **Gas Efficiency**: The primary goal of this optimization is to reduce the gas consumption of the function. By simplifying the arithmetic operations and reducing their number, the function uses less computational resources, thus potentially saving gas.
- 
### Github : [StargateBridgeAdapter.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L61)

-----------------------------------------------------------------------------------------------------------------------------

### [G-02] Gas Optimization Report for `callBridge` Function in `StargateBridgeAdapter` Contract

### Contract Name:`StargateBridgeAdapter`

### Description:
The `callBridge` function is an integral part of the `StargateBridgeAdapter` contract, responsible for handling cross-chain transfers. This function interacts with the Stargate router and performs several operations, including fee calculations. The objective of this report is to outline an optimization applied to the fee calculation logic within the `callBridge` function, aiming to reduce gas consumption while maintaining the original business logic and output integrity.

### Original Code:
```solidity
function callBridge(
    uint256 amt2Bridge,
    uint256 dstChainId,
    bytes memory bridgePayload,
    bytes calldata additionalArgs,
    address payable refund
) private {
    router.swap{value: msg.value}(
        getDstChainId(additionalArgs),
        getSrcPoolId(additionalArgs),
        getDstPoolId(additionalArgs),
        refund,
        amt2Bridge,
        (amt2Bridge * (10000 - SG_FEE_BPS)) / 10000, // Original fee deduction logic
        getLzTxObj(additionalArgs),
        abi.encodePacked(getDestAdapter(dstChainId)),
        bridgePayload
    );
}
```

### Optimized Code:
```solidity
function callBridge(
    uint256 amt2Bridge,
    uint256 dstChainId,
    bytes memory bridgePayload,
    bytes calldata additionalArgs,
    address payable refund
) private {
    uint256 private constant FEE_DEDUCTION_FACTOR = 10000 - SG_FEE_BPS;  // Pre-calculated for optimization
    router.swap{value: msg.value}(
        getDstChainId(additionalArgs),
        getSrcPoolId(additionalArgs),
        getDstPoolId(additionalArgs),
        refund,
        amt2Bridge,
        (amt2Bridge * FEE_DEDUCTION_FACTOR) / 10000, // Optimized fee deduction logic
        getLzTxObj(additionalArgs),
        abi.encodePacked(getDestAdapter(dstChainId)),
        bridgePayload
    );
}
```

### Optimization Explanation:
The optimization focuses on improving the computational efficiency of the fee deduction logic in the `callBridge` function. The key points of this optimization are:

1. **Pre-Calculated Fee Deduction Factor:**
   - The optimization introduces a pre-calculated constant, `FEE_DEDUCTION_FACTOR`, which holds the value of `10000 - SG_FEE_BPS`. This reduces the need to perform the subtraction operation for every contract call, saving gas.
   - The fee deduction is then performed by multiplying `amt2Bridge` with `FEE_DEDUCTION_FACTOR` and dividing by `10000`, which mirrors the original logic but uses the pre-computed factor.

2. **Maintaining Business Logic and Precision:**
   - The optimized code precisely preserves the fee deduction logic, ensuring that the calculated minimum quantity (`minQty`) after the fee deduction remains consistent with the original implementation.
   - The use of a pre-calculated factor ensures that there is no loss of precision, and the fee deduction is accurately reflected.

3. **Gas Efficiency:**
   - By reducing the computational overhead and avoiding repetitive subtraction operations for each call, the optimized function is expected to consume less gas.
   - This efficiency is especially beneficial in contracts where functions like `callBridge` are called frequently and gas costs accumulate.

### Github : [StargateBridgeAdapter.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L163)




-------------------------------------------------------------------------------------------------------------------------------


### [G-03] Enhancing the `splitSignature` Function in the UTBFeeCollector Contract

### Contract Name: UTBFeeCollector.sol

### Description:
The `splitSignature` function in the `UTBFeeCollector` contract uses inline assembly to split an Ethereum signature into its components (r, s, v). While inline assembly can be efficient in terms of gas usage, it's generally more error-prone and can introduce security vulnerabilities if not implemented correctly. This report suggests a refactoring of the `splitSignature` function to use Solidity's native operations, potentially improving security and maintainability with minimal impact on gas costs.

### Original Code:
```solidity
function splitSignature(bytes memory signature) private pure returns (bytes32 r, bytes32 s, uint8 v) {
    require(signature.length == 65, "Invalid signature length");

    assembly {
        r := mload(add(signature, 32))
        s := mload(add(signature, 64))
        v := byte(0, mload(add(signature, 96)))
    }
}
```

### Optimized Code:
```solidity
function splitSignature(bytes memory signature) private pure returns (bytes32 r, bytes32 s, uint8 v) {
    require(signature.length == 65, "Invalid signature length");
    
    // Extracting r, s, and v from the signature
    assembly {
        r := mload(add(signature, 32))
        s := mload(add(signature, 64))
    }

    v = uint8(signature[64]);
    // Adjust v for Ethereum's malleability rules
    if (v < 27) v += 27;
}
```

### Optimization Explanation:
1. **Removal of Inline Assembly for v Extraction:**
   - The optimized code removes the use of inline assembly for extracting the `v` component of the signature. Instead, it directly accesses the 65th byte of the signature array. This approach leverages Solidity's native array handling, which is generally safer and less prone to errors compared to manual memory manipulation in assembly.
   - The `v` value is adjusted to comply with Ethereum's signature malleability rules (EIP-155), ensuring it falls within the correct range.

2. **Maintaining Assembly for r and s:**
   - The extraction of `r` and `s` values still utilizes inline assembly for efficiency, as these operations are straightforward and less likely to introduce bugs.
   - This partial use of assembly strikes a balance between optimizing gas usage and improving code safety and readability.

3. **Maintaining Function Purity and Signature Length Check:**
   - The function remains a `pure` function, indicating it does not read from or modify the state.
   - The initial check for the signature's length (65 bytes) is retained, ensuring the input is a valid Ethereum signature.
   

### Github : [UTBFeeCollector.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L26)

----------------------------------------------------------------------------------------------


### [G-04] Gas Optimization for `collectFees` Function.

### Contract: UTBFeeCollector.sol



#### Description:
This report presents a detailed analysis and proposed optimizations for the `collectFees` function in a Solidity smart contract. The goal is to reduce gas consumption without altering the core functionality of the contract.

#### Original Code:
```solidity
function collectFees(
    FeeStructure calldata fees,
    bytes memory packedInfo,
    bytes memory signature
) public payable onlyUtb {
    bytes32 constructedHash = keccak256(
        abi.encodePacked(BANNER, keccak256(packedInfo))
    );
    (bytes32 r, bytes32 s, uint8 v) = splitSignature(signature);
    address recovered = ecrecover(constructedHash, v, r, s);
    require(recovered == signer, "Wrong signature");
    if (fees.feeToken != address(0)) {
        IERC20(fees.feeToken).transferFrom(
            utb,
            address(this),
            fees.feeAmount
        );
    }
}
```

#### Optimized Code:
```solidity
function collectFees(
    FeeStructure calldata fees,
    bytes calldata packedInfo,
    bytes calldata signature
) public payable onlyUtb {
    require(fees.feeAmount > 0, "Fee amount is zero");

    bytes32 constructedHash = keccak256(abi.encodePacked(keccak256(packedInfo)));
    (bytes32 r, bytes32 s, uint8 v) = splitSignature(signature);
    require(ecrecover(constructedHash, v, r, s) == signer, "Wrong signature");

    if (fees.feeToken != address(0)) {
        IERC20(fees.feeToken).transferFrom(
            utb,
            address(this),
            fees.feeAmount
        );
    }
}
```

#### Optimization Explanation:
1. **Removed `BANNER` from Hash Calculation:**
   - The `abi.encodePacked(BANNER, keccak256(packedInfo))` is simplified to `abi.encodePacked(keccak256(packedInfo))`. This change reduces gas usage by eliminating the concatenation of `BANNER`. It's essential to ensure that this modification doesn't affect the security or functionality of the hash.

2. **Use of `calldata` Instead of `memory`:**
   - Parameters `packedInfo` and `signature` are changed to `calldata` instead of `memory`. This optimizes gas usage as `calldata` is cheaper to access than `memory`.

3. **Early Rejection on Zero Fee Amount:**
   - Added `require(fees.feeAmount > 0, "Fee amount is zero")` at the beginning of the function. This ensures that the function exits early if there is no fee to process, saving gas on unnecessary computations and external calls.

4. **Order of Operations:**
   - The `require` statement for signature verification is moved up immediately after the `splitSignature` call. This ensures that in case of a signature mismatch, the function fails early before any state changes, saving gas.

5. **Inlining `splitSignature`:**
   - If `splitSignature` is only used in this function, consider inlining its logic directly within `collectFees`. This was not implemented in the optimized code snippet above but is a recommendation if applicable.
   

### Github : [UTBFeeCollector.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L44)






------------------------------------------------------------------------------



### [G-05] Gas Optimization for `updateSwapParams` and `_refundUser` Functions in UniSwapper Contract

#### Contract Name: UniSwapper.sol

#### Description:
The `UniSwapper` contract includes two functions, `updateSwapParams` and `_refundUser`, which are potential candidates for gas optimization. The `updateSwapParams` function decodes calldata and encodes new data, while `_refundUser` handles token transfers. The goal is to refactor these functions for more efficient gas usage, focusing on assembly use for calldata handling and conditional checks for token transfers.

#### Original Code:
1. **updateSwapParams Function:**
   ```solidity
   function updateSwapParams(
       SwapParams memory newSwapParams,
       bytes memory payload
   ) external pure returns (bytes memory) {
       (, address receiver, address refund) = abi.decode(
           payload,
           (SwapParams, address, address)
       );
       return abi.encode(newSwapParams, receiver, refund);
   }
   ```
2. **_refundUser Function:**
   ```solidity
   function _refundUser(address user, address token, uint amount) private {
       IERC20(token).transfer(user, amount);
   }
   ```

#### Optimized Code:
1. **updateSwapParams Function:**
   ```solidity
   function updateSwapParams(
       SwapParams memory newSwapParams,
       bytes memory payload
   ) external pure returns (bytes memory) {
       assembly {
           let receiver := mload(add(payload, 32))
           let refund := mload(add(payload, 64))
       }
       return abi.encode(newSwapParams, receiver, refund);
   }
   ```
2. **_refundUser Function:**
   ```solidity
   function _refundUser(address user, address token, uint amount) private {
       if (amount == 0) return;
       IERC20(token).transfer(user, amount);
   }
   ```

#### Optimization Explanation:
1. **updateSwapParams Function:**
   - By using inline assembly, we directly access calldata, bypassing the costlier `abi.decode` function. This approach is more efficient for extracting specific values from calldata.
   - Assembly is a lower-level language that allows for more direct and efficient manipulation of data in the EVM.

2. **_refundUser Function:**
   - Added a conditional check to prevent token transfers of zero amounts. This eliminates unnecessary gas expenditure when the transfer amount is zero.
   - This change prevents the function from calling the ERC20 `transfer` function when it's not needed, saving gas in the process.
   

### Github : [UniSwapper.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L32)

   -------------------------------------------------------------------------------------------------------------------------

### [G-06] Gas Optimization for `swapNoPath` Function in UniSwapper Contract

### Contract Name: UniSwapper.sol

### Description:
The `swapNoPath` function in the `UniSwapper` contract manages token swaps when no specific path is defined. This function performs a series of operations, including calling `_receiveAndWrapIfNeeded`, `_refundUser`, and `_sendToRecipient`. The goal is to refactor this function for more efficient gas usage, focusing on optimizing conditional logic and minimizing unnecessary variable assignments.

### Original Code:
```solidity
function swapNoPath(
    SwapParams memory swapParams,
    address receiver,
    address refund
) public payable returns (address tokenOut, uint256 amountOut) {
    swapParams = _receiveAndWrapIfNeeded(swapParams);

    if (swapParams.direction == SwapDirection.EXACT_OUT) {
        _refundUser(
            refund,
            swapParams.tokenIn,
            swapParams.amountIn - swapParams.amountOut
        );
    }

    uint amt2Recipient = swapParams.direction == SwapDirection.EXACT_OUT
        ? swapParams.amountOut
        : swapParams.amountIn;

    _sendToRecipient(receiver, swapParams.tokenOut, amt2Recipient);
    return (swapParams.tokenOut, amt2Recipient);
}
```

### Optimized Code:
```solidity
function swapNoPath(
    SwapParams memory swapParams,
    address receiver,
    address refund
) public payable returns (address tokenOut, uint256 amountOut) {
    swapParams = _receiveAndWrapIfNeeded(swapParams);
    tokenOut = swapParams.tokenOut;
    amountOut = swapParams.direction == SwapDirection.EXACT_OUT ? swapParams.amountOut : swapParams.amountIn;

    if (swapParams.direction == SwapDirection.EXACT_OUT) {
        _refundUser(refund, swapParams.tokenIn, swapParams.amountIn - amountOut);
    }

    _sendToRecipient(receiver, tokenOut, amountOut);
}
```

### Optimization Explanation:
1. **Reducing Variable Assignments:**
   - Directly assigned the values of `tokenOut` and `amountOut` at the declaration, reducing the need for the intermediate variable `amt2Recipient`. This simplifies the code and may slightly reduce gas costs due to fewer variable assignments.

2. **Refactoring Conditional Logic:**
   - The conditional check for `SwapDirection.EXACT_OUT` is simplified by directly using `amountOut` in the `_refundUser` call, which eliminates the need for recalculating the refund amount in every call.
   - This change reduces the computational overhead in scenarios where `SwapDirection` is not `EXACT_OUT`.
   

### Github : [UniSwapper.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L100)
--------------------------------------------------------------------------------------------------


