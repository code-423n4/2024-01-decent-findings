## UTB contract

### Change State Variables to Immutable
- **Problem**: Mutable storage variables like `executor`, `feeCollector`, `wrapped` can be more gas-intensive.
- **Mitigation**:
  ```solidity
  IUTBExecutor immutable executor;
  IUTBFeeCollector immutable feeCollector;
  IWETH immutable wrapped;
  ```
- **Impact**: Reduces gas costs for reads and ensures these values cannot be altered after contract deployment.

### Use Private and External for Getters
- **Problem**: `public` getters create automatic getter functions, which can be less efficient.
- **Mitigation**:
  ```solidity
  mapping(uint8 => address) private swappers;
  function getSwapper(uint8 id) external view returns (address) { return swappers[id]; }
  ```
- **Impact**: Reduces gas cost and external call overhead.

###  Batch ERC20 Transfer and Approval
- **Problem**: Separate ERC20 `transfer` and `approve` calls in `retrieveAndCollectFees` are inefficient.
- **Mitigation**: Implement a multicall pattern or use contracts like DSProxy to batch these calls.
- **Impact**: Saves gas by reducing the number of transactions.

### Use Transfer Instead of TransferFrom
- **Problem**: `transferFrom` in `retrieveAndCollectFees` requires an approval step.
- **Mitigation**:
  ```solidity
  feeCollector.collectFees{value: msg.value}(fees, packedInfo, signature);
  ```
- **Impact**: Shifts gas costs to the user and simplifies the process.

### Use Unchecked Math
- **Problem**: Safe math checks in `performSwap` can be redundant and gas-intensive.
- **Mitigation**:
  ```solidity
  unchecked {
      // Perform calculations
  }
  ```
- **Impact**: Reduces gas cost, but requires careful validation to avoid overflows/underflows.

### Use Events Instead of Return Data
- **Problem**: Returning data in `callBridge` can be gas-intensive.
- **Mitigation**: Emit events with necessary data instead of returning it.
- **Impact**: Saves gas and streamlines the interface.

### Validate Data Hashes Instead of Signatures
- **Problem**: Signature verification on-chain is expensive.
- **Mitigation**: Move signature verification off-chain and validate data hashes on-chain.
- **Impact**: Significantly reduces gas costs, but requires off-chain infrastructure.

### Add Pagination / Limit Payload Size
- **Problem**: Large `payload` data can cause out-of-gas errors.
- **Mitigation**: Implement pagination or limit the size of `payload`.
- **Impact**: Ensures consistent gas costs and avoids failures due to large data.

### Use Interface for `executor`
- **Problem**: Using a concrete contract for `executor` limits flexibility.
- **Mitigation**: Use an interface for better abstraction.
- **Impact**: Increases contract flexibility and potential for future upgrades.


## `UTBExecutor` contract 

### Use `calldata` for Large Parameters
- **Problem**: Using `memory` for large parameters like `payload` increases gas costs due to memory copying.
- **Mitigation**:
  ```solidity
  function execute(
      address target,
      address paymentOperator,
      bytes calldata payload,
      address tokenOut,
      uint256 amountOut,
      address payable refund
  ) external { ... }
  ```
- **Impact**: Reduces gas costs by avoiding unnecessary memory allocation and copying.

### Immutable Storage for Owner
- **Problem**: Mutable storage variables for frequently accessed values like `owner` increase gas costs.
- **Mitigation**:
  ```solidity
  address immutable owner;
  constructor(address _owner) {
      owner = _owner;
  }
  ```
- **Impact**: Decreases read cost of the `owner` variable, improving gas efficiency.

### Private `onlyOwner` Modifier
- **Problem**: Public modifiers create additional overhead.
- **Mitigation**:
  ```solidity
  modifier onlyOwner() {
      require(msg.sender == owner, "Not owner");
      _;
  }
  ```
- **Impact**: Slightly reduces gas cost.

### Use `transfer` Instead of `transferFrom`
- **Problem**: `transferFrom` requires two steps: approval and transfer, increasing gas usage.
- **Mitigation**: This optimization is more applicable to ERC20 tokens than ETH transfers. For ETH, `transfer` is already used.

### Batch Transfer and Approve Calls
- **Problem**: Separate transfer and approve calls are inefficient.
- **Mitigation**: Implement a multicall pattern or use contracts like DSProxy to batch these calls.
- **Impact**: Saves gas by reducing the number of transactions.

### Unchecked Math for Balances
- **Problem**: Safe math checks can be redundant and gas-intensive.
- **Mitigation**:
  ```solidity
  unchecked {
      // Perform balance calculations
  }
  ```
- **Impact**: Reduces gas cost, but requires careful validation to avoid overflows/underflows.


### Separate Utility Function for Token Logic
- **Problem**: Code duplication for token transfer and approve logic.
- **Mitigation**:
  ```solidity
  function transferToken(address token, address to, uint256 amount) internal { ... }
  ```
- **Impact**: Reduces code duplication and improves maintainability.


### Use `calldata` for Large Parameters

#### Problematic Code:
```solidity
function execute(
    address target,
    address paymentOperator,
    bytes memory payload,
    // Other parameters
) public { ... }
```

#### Mitigation:
```solidity
function execute(
    address target,
    address paymentOperator,
    bytes calldata payload,
    // Other parameters
) public { ... }
```

### Use Immutable for Owner

#### Problematic Code:
```solidity
address owner;
```

#### Mitigation:
```solidity
address immutable owner;
constructor(address _owner) {
    owner = _owner;
}
```

### Private `onlyOwner` Modifier

#### Problematic Code:
```solidity
modifier onlyOwner() public {
    require(msg.sender == owner, "Not owner");
    _;
}
```

#### Mitigation:
```solidity
modifier onlyOwner() private {
    require(msg.sender == owner, "Not owner");
    _;
}
```

### Batch Transfer and Approve Calls

*Note*: If your contract performs separate ERC20 `transfer` and `approve` calls, consider batching them. This requires a more complex change involving contract architecture rather than a direct code snippet change.

### Unchecked Math for Balances

#### Problematic Code:
```solidity
uint256 newBalance = balance + amount;
```

#### Mitigation:
```solidity
unchecked {
    uint256 newBalance = balance + amount;
}
```

### Separate Utility Function for Token Logic

#### Problematic Code:
```solidity
IERC20(token).transfer(recipient, amount);
IERC20(token).approve(spender, amount);
```

#### Mitigation:
```solidity
function transferToken(address token, address recipient, uint256 amount) internal {
    IERC20(token).transfer(recipient, amount);
}
function approveToken(address token, address spender, uint256 amount) internal {
    IERC20(token).approve(spender, amount);
}
```

### Use Assembly/Raw Call for External Calls

#### Problematic Code:
```solidity
targetContract.call(payload);
```

#### Mitigation:
Use low-level `call` with assembly for critical external calls. Note this increases complexity and security risk.

### Add Guardrails for Execute

#### Problematic Code:
```solidity
function execute(...) public { ... }
```

#### Mitigation:
Implement checks on token/ETH amounts within the `execute` function.

```solidity
function execute(...) public {
    require(amount <= maxAllowedAmount, "Amount exceeds limit");
    ...
}
```

## `UTBFeeCollector` 

### Use `calldata` for Large Parameters

#### Problematic Code:
```solidity
function collectFees(
    FeeStructure calldata fees,
    bytes memory packedInfo,
    bytes memory signature
) public payable onlyUtb { ... }
```

#### Mitigation:
```solidity
function collectFees(
    FeeStructure calldata fees,
    bytes calldata packedInfo,
    bytes calldata signature
) public payable onlyUtb { ... }
```



### Separate Utility Function for Token Logic

#### Problematic Code:
```solidity
IERC20(fees.feeToken).transferFrom(utb, address(this), fees.feeAmount);
```

#### Mitigation:
```solidity
function transferToken(address token, uint256 amount) internal {
    IERC20(token).transferFrom(utb, address(this), amount);
}
```


### Event for Execution Success/Failure

#### Problematic Code:
No events for execution results.

#### Mitigation:
```solidity
event FeesCollected(address token, uint256 amount);
event FeesRedeemed(address token, uint256 amount);

// Emit these events in the respective functions.
```


### Add Guardrails for Execute

The `redeemFees` function should include checks to prevent redeeming more than the available balance.

#### Problematic Code:
```solidity
function redeemFees(address token, uint amount) public onlyOwner { ... }
```

#### Mitigation:
```solidity
function redeemFees(address token, uint amount) public onlyOwner {
    require(amount <= getBalance(token), "Amount exceeds balance");
    ...
}

function getBalance(address token) public view returns (uint) {
    if (token == address(0)) {
        return address(this).balance;
    } else {
        return IERC20(token).balanceOf(address(this));
    }
}
```

### Use a More Efficient Signature Verification Method

#### Problematic Code:
```solidity
function collectFees(
    FeeStructure calldata fees,
    bytes memory packedInfo,
    bytes memory signature
) public payable onlyUtb {
    // ...signature verification logic...
}
```

#### Mitigation:
The current signature verification process involves splitting the signature and using `ecrecover`. This process is quite standard, but you could optimize it further by validating signatures off-chain and passing the recovered address as a parameter. However, this would reduce decentralization and trustlessness, as it relies on off-chain computations.





## `UniSwapper` contract

### Use `calldata` for Large Parameters

#### Problematic Code:
```solidity
function updateSwapParams(
    SwapParams memory newSwapParams,
    bytes memory payload
) external pure returns (bytes memory) { ... }
```

#### Mitigation:
```solidity
function updateSwapParams(
    SwapParams calldata newSwapParams,
    bytes calldata payload
) external pure returns (bytes memory) { ... }
```

### Immutable Storage for Owner

#### Problematic Code:
Inheriting `UTBOwned`, the `owner` variable should be immutable if it's not already.

#### Mitigation:
In `UTBOwned` contract:
```solidity
address immutable owner;
constructor() {
    owner = msg.sender;
}
```



### Use Assembly/Raw Call for External Calls

#### Problematic Code:
```solidity
IV3SwapRouter(uniswap_router).exactInput(params);
IV3SwapRouter(uniswap_router).exactOutput(params);
```

#### Mitigation:
Consider using low-level `call` for these interactions, but be cautious as it increases complexity and security risk.


##  `DecentEthRouter` contract 

### Use `calldata` for Large Parameters

#### Problematic Code:
```solidity
function bridgeWithPayload(
    uint16 _dstChainId,
    address _toAddress,
    uint _amount,
    bool deliverEth,
    uint64 _dstGasForCall,
    bytes memory additionalPayload
) public payable { ... }
```

#### Mitigation:
```solidity
function bridgeWithPayload(
    uint16 _dstChainId,
    address _toAddress,
    uint _amount,
    bool deliverEth,
    uint64 _dstGasForCall,
    bytes calldata additionalPayload
) public payable { ... }
```

### Immutable Storage for Owner

#### Problematic Code:
Inherited from `Owned`, the `owner` should be immutable.

#### Mitigation:
In `Owned` contract:
```solidity
address immutable owner;
constructor() {
    owner = msg.sender;
}
```


### Unchecked Math for Balances

#### Problematic Code:
```solidity
balanceOf[msg.sender] += amount;
```

#### Mitigation:
```solidity
unchecked {
    balanceOf[msg.sender] += amount;
}
```

### Off-Chain Address(0) Checks

Validate `address(0)` checks off-chain where possible, such as in `registerDcntEth` and `addDestinationBridge`.

### Separate Utility Function for Token Logic

#### Problematic Code:
```solidity
weth.transfer(_to, _amount);
```

#### Mitigation:
```solidity
function transferToken(address token, address to, uint256 amount) private {
    IERC20(token).transfer(to, amount);
}
```


### Break Functions into Smaller Internal Functions

#### Problematic Code:
Large functions like `bridgeWithPayload` and `onOFTReceived`.

#### Mitigation:
Refactor these functions into smaller, more focused internal functions.

### Event for Execution Success/Failure

#### Problematic Code:
Lack of events for tracking important operations.

#### Mitigation:
```solidity
event BridgeOperation(address indexed to, uint amount, uint16 dstChainId);
// Emit this event in relevant functions.
```

### Add Guardrails for Execute

#### Problematic Code:
In functions like `removeLiquidityEth` and `removeLiquidityWeth`, ensure limits or conditions are in place to prevent abuse or errors.

#### Mitigation:
```solidity
function removeLiquidityEth(uint256 amount) public onlyEthChain userIsWithdrawing(amount) {
    require(amount <= MAX_AMOUNT, "Amount exceeds limit");
    // Rest of the function
}
```


## `DecentEthRouter` contract

### Use `unchecked` for Operations That Will Not Overflow

#### Problematic Code:
```solidity
balanceOf[msg.sender] += amount;
```

#### Mitigation:
```solidity
unchecked {
    balanceOf[msg.sender] += amount;
}
```

### Use `calldata` Instead of `memory` for Function Arguments That Do Not Get Mutated

#### Problematic Code:
```solidity
function bridgeWithPayload(
    uint16 _dstChainId,
    address _toAddress,
    uint _amount,
    bool deliverEth,
    uint64 _dstGasForCall,
    bytes memory additionalPayload
) public payable { ... }
```

#### Mitigation:
```solidity
function bridgeWithPayload(
    uint16 _dstChainId,
    address _toAddress,
    uint _amount,
    bool deliverEth,
    uint64 _dstGasForCall,
    bytes calldata additionalPayload
) public payable { ... }
```

### Use Custom Errors

#### Problematic Code:
```solidity
require(gasCurrencyIsEth, "Gas currency is not ETH");
```

#### Mitigation:
```solidity
error GasCurrencyNotETH();

...

if (!gasCurrencyIsEth) revert GasCurrencyNotETH();
```

### Don't Initialize Variables With Default Values

#### Problematic Code:
```solidity
bool public gasCurrencyIsEth; // default is false
```

#### Mitigation:
```solidity
// Remove default value if not necessary
bool public gasCurrencyIsEth;
```

### Use Minimal Visibility for Functions and State Variables

#### Problematic Code:
```solidity
uint8 public constant MT_ETH_TRANSFER = 0;
```

#### Mitigation:
```solidity
uint8 private constant MT_ETH_TRANSFER = 0;
```

### Use Shorter Variable Types

#### Problematic Code:
```solidity
mapping(address => uint256) public balanceOf;
```

#### Mitigation:
```solidity
// If balances will not be very high, consider using a smaller type
mapping(address => uint128) public balanceOf;
```

### Use External Instead of Public for Functions Never Called in Contract

#### Problematic Code:
```solidity
function registerDcntEth(address _addr) public onlyOwner { ... }
```

#### Mitigation:
```solidity
function registerDcntEth(address _addr) external onlyOwner { ... }
```

### Extract Invocations into Internal Functions to Save on Call Overhead

#### Problematic Code:
```solidity
weth.transfer(_to, _amount);
```

#### Mitigation:
```solidity
function _transferWETH(address to, uint256 amount) private {
    weth.transfer(to, amount);
}
```

### Be Mindful of Data Location - Memory Is Cheaper Than Storage

#### Problematic Code:
```solidity
uint256 balance = balanceOf[msg.sender];
```

#### Mitigation:
```solidity
// Use directly from storage if only read once
require(balanceOf[msg.sender] >= amount, "not enough balance");
```

### Use Immutable Instead of Constant for Variables That Never Change

#### Problematic Code:
Variables initialized in the constructor should be marked as immutable if they don't change.

#### Mitigation:
```solidity
IWETH public immutable weth;
```

### Use Private Visibility and Getters Instead of Public

#### Problematic Code:
```solidity
mapping(uint16 => address) public destinationBridges;
```

#### Mitigation:
```solidity
mapping(uint16 => address) private destinationBridges;

function getDestinationBridge(uint16 chainId) external view returns (address) {
    return destinationBridges[chainId];
}
```

### Set Function Mutability to View/Pure When Possible

#### Problematic Code:
```solidity
function getId() public pure returns (uint8) { ... }
```

#### Mitigation:
```solidity
function getId() external pure returns (uint8) { ... }
```

## `StargateBridgeAdapter` 

### Use `unchecked` for Operations That Will Not Overflow

#### Problematic Code:
Not directly applicable as there are no evident arithmetic operations that clearly won't overflow in the contract.

### Cache State Variables in Stack Variables

#### Problematic Code:
```solidity
IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);
```

#### Mitigation:
```solidity
address _bridgeToken = bridgeToken;
IERC20(_bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);
```

### Use `calldata` Instead of `memory` for Immutable Function Arguments

#### Problematic Code:
```solidity
function getBridgeToken(bytes calldata additionalArgs) external pure returns (address bridgeToken) { ... }
```

#### Mitigation:
Already optimized using `calldata`.

### Use Custom Errors

#### Problematic Code:
```solidity
require(dstAddr != address(0), "dst chain address not set");
```

#### Mitigation:
```solidity
error DestinationChainAddressNotSet();

...

if (dstAddr == address(0)) revert DestinationChainAddressNotSet();
```

### Don't Initialize Variables With Default Values

#### Problematic Code:
Not directly applicable as the contract doesn't initialize state variables with default values unnecessarily.

### Mark Functions as Payable If They Always Revert for Normal Users

#### Problematic Code:
Not directly applicable as there are no functions that always revert and require being marked as payable.

### Using Private Rather Than Public for Constants

#### Problematic Code:
```solidity
uint8 public constant BRIDGE_ID = 1;
```

#### Mitigation:
```solidity
uint8 private constant BRIDGE_ID = 1;
```

### Use `!= 0` Instead of `> 0` for Unsigned Integer Comparison

#### Problematic Code:
Not directly applicable as there are no evident comparisons of this nature in the contract.

### Avoid Unnecessary Data Copies Between Memory and Storage

#### Problematic Code:
Optimization already applied as no unnecessary data copies are evident.

### Declare Constant Variables as Immutable Rather Than Constant

#### Problematic Code:
```solidity
uint8 public constant BRIDGE_ID = 1;
```

#### Mitigation:
```solidity
uint8 public immutable BRIDGE_ID;
constructor() {
    BRIDGE_ID = 1;
}
```

### Use Minimal Visibility for Functions and State Variables

#### Problematic Code:
```solidity
function getId() public pure returns (uint8) { ... }
```

#### Mitigation:
```solidity
function getId() external pure returns (uint8) { ... }
```


### Use External Instead of Public for Functions Never Called Internally

#### Problematic Code:
```solidity
function getId() public pure returns (uint8) { ... }
```

#### Mitigation:
```solidity
function getId() external pure returns (uint8) { ... }
```

### Extract Invocations into Internal Functions to Save on Call Overhead

#### Problematic Code:
```solidity
router.swap{value: msg.value}(...);
```

#### Mitigation:
```solidity
function _routerSwap(...) private {
    router.swap{value: msg.value}(...);
}
```

### Be Mindful of Data Location - Memory Is Cheaper Than Storage

#### Problematic Code:
State variables are accessed multiple times in functions like `callBridge`.

#### Mitigation:
Cache state variables in memory or stack variables.

### Use Immutable Instead of Constant for Variables That Never Change

#### Problematic Code:
```solidity
uint8 public constant BRIDGE_ID = 1;
```

#### Mitigation:
```solidity
uint8 public immutable BRIDGE_ID;
constructor() {
    BRIDGE_ID = 1;
}
```

### Use Private Visibility and Getters Instead of Public

#### Problematic Code:
```solidity
uint8 public constant BRIDGE_ID = 1;
```

#### Mitigation:
```solidity
uint8 private constant BRIDGE_ID = 1;
function getBridgeId() external pure returns (uint8) {
    return BRIDGE_ID;
}
```

### Use `calldata` for External Functions With Large Parameters

#### Problematic Code:
```solidity
function bridge(
    uint256 amt2Bridge,
    SwapInstructions memory postBridge,
    // other

 parameters
) public payable onlyUtb returns (bytes memory bridgePayload) { ... }
```

#### Mitigation:
```solidity
function bridge(
    uint256 amt2Bridge,
    SwapInstructions calldata postBridge,
    // other parameters
) public payable onlyUtb returns (bytes memory bridgePayload) { ... }
```

### Set Function Mutability to View/Pure When Possible

#### Problematic Code:
```solidity
function getId() public pure returns (uint8) { ... }
```

#### Mitigation:
This is already optimized as a pure function.

## `DecentBridgeAdapter` contract

### Avoid Unnecessary Data Copies Between Memory and Storage

#### Problematic Code:
Repetitive access to state variables like `router` and `bridgeToken` in functions.

#### Mitigation:
Cache state variables in local memory or stack variables where frequently accessed.

### Declare Constant Variables as Immutable

#### Problematic Code:
```solidity
uint8 public constant BRIDGE_ID = 0;
```

#### Mitigation:
```solidity
uint8 public immutable BRIDGE_ID;
constructor() {
    BRIDGE_ID = 0;
}
```

### Batch External Calls

#### Problematic Code:
Separate calls to `IERC20(bridgeToken).transferFrom` and `IERC20(bridgeToken).approve` in `bridge` function.

#### Mitigation:
Consider batch calls or multicall patterns to combine these operations.


### Use Minimal Visibility for Functions and State Variables

#### Problematic Code:
```solidity
function getId() public pure returns (uint8) { ... }
```

#### Mitigation:
```solidity
function getId() external pure returns (uint8) { ... }
```

### Use Shorter Variable Types

#### Problematic Code:
`uint256` is used for `dstChainId` which may not require 256 bits.

#### Mitigation:
Use a smaller data type if possible, like `uint16` or `uint8`.

### Use External Instead of Public for Functions Never Called Internally

#### Problematic Code:
```solidity
function getId() public pure returns (uint8) { ... }
```

#### Mitigation:
```solidity
function getId() external pure returns (uint8) { ... }
```

### Extract Invocations into Internal Functions

#### Problematic Code:
Operations in `bridge` function.

#### Mitigation:
Refactor complex or repetitive operations into internal functions.

### Be Mindful of Data Location

#### Problematic Code:
Repeatedly decoding `SwapParams` from `postBridge.swapPayload`.

#### Mitigation:
Decode once and pass as a parameter to internal functions if needed.

### Use Immutable Instead of Constant

#### Problematic Code:
```solidity
uint8 public constant BRIDGE_ID = 0;
```

#### Mitigation:
```solidity
uint8 public immutable BRIDGE_ID;
constructor() {
    BRIDGE_ID = 0;
}
```

### Set Function Mutability to View/Pure When Possible

#### Problematic Code:
```solidity
function getId() public pure returns (uint8) { ... }
```

#### Mitigation:
Already optimized.

### Use Private Visibility and Getters Instead of Public

#### Problematic Code:
```solidity
uint8 public constant BRIDGE_ID = 0;
```

#### Mitigation:
```solidity
uint8 private constant BRIDGE_ID = 0;
function getBridgeId() external pure returns (uint8) {
    return BRIDGE_ID;
}
```

### Use `calldata` for External Functions With Large Parameters

#### Problematic Code:
```solidity
function bridge(
    uint256 amt2Bridge,
    SwapInstructions memory postBridge,
    // Other parameters
) public payable onlyUtb returns (bytes memory bridgePayload) { ... }
```

#### Mitigation:
```solidity
function bridge(
    uint256 amt2Bridge,
    SwapInstructions calldata postBridge,
    // Other parameters
) public payable onlyUtb returns (bytes memory bridgePayload) { ... }
```