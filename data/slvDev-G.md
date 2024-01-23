## Gas Findings

|    | Issue | Instances | Total Gas Saved |
|----|-------|:---------:|:---------:|
| [G-01] | Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct` | 6 | - |
| [G-02] | Stack variable is only used once | 5 | 48 |
| [G-03] | Optimize `require/revert` Statements with Assembly | 12 | 3600 |
| [G-04] | Use Cached Contracts for Multiple External Calls | 17 | 2576 |
| [G-05] | Optimize Gas by Using Only Named Returns | 14 | 616 |
| [G-06] | Use bytes32 in place of string | 2 | 0 |
| [G-07] | Use Assembly for Hash Calculations | 2 | 2010 |
| [G-08] | Avoid Inverting `if`-`else` Conditions | 2 | 6 |
| [G-09] | Optimize External Calls with Assembly for Memory Efficiency | 14 | 457 |
| [G-10] | Unlimited gas consumption risk due to external call recipients | 5 | - |
| [G-11] | Reduce deployment costs by tweaking contracts' metadata | 11 | 116600 |
| [G-12] | Unused `private` functions should be removed to save deployment gas | 1 | 0 |
| [G-13] | Use `revert()` to gain maximum gas savings | 12 | 600 |
| [G-14] | Optimize Ether Transfers with `receive()` Function | 13 | 585 |
| [G-15] | Avoid contract existence checks by using low-level calls | 47 | 4700 |
| [G-16] | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 31 | 198 |


## Gas Findings Details

### [G-01] Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`

For `destinationBridgeAdapter` and `lzIdLookup` used `dstChainId` as a key in `registerRemoteBridgeAdapter()` function so it can be combined into a single `mapping` of an `address`/ID to a `struct` to save gas.

<details>
<summary><i>4 issue instances in 2 files:</i></summary>


```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

14: mapping(uint256 => address) public destinationBridgeAdapter
16: mapping(uint256 => uint16) lzIdLookup
```
[14](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L14) | [16](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L16)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

25: mapping(uint256 => address) public destinationBridgeAdapter
26: mapping(uint256 => uint16) lzIdLookup
```
[25](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L25) | [26](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L26)
</details>


### [G-02] Stack variable is only used once

If the variable is only accessed once, it's cheaper to use the assigned value directly that one time, and save the 3 gas the extra stack assignment would spend

<details>
<summary><i>5 issue instances in 2 files:</i></summary>

```solidity
File: src/UTB.sol

/// @audit - `native` variable
287: bool native = approveAndCheckIfNative(instructions, amt2Bridge)
/// @audit - `s` variable
326: ISwapper s = ISwapper(swapper)
/// @audit - `b` variable
335: IBridgeAdapter b = IBridgeAdapter(bridge)
```
[287](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L287) | [326](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L326) | [335](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L335)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

/// @audit - `GAS_FOR_RELAY` variable
96: uint256 GAS_FOR_RELAY = 100000
/// @audit - `gasAmount` variable
97: uint256 gasAmount = GAS_FOR_RELAY + _dstGasForCall
/// @audit - `callParams` variable
```
[96](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L96) | [97](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L97)
</details>


### [G-03] Optimize `require/revert` Statements with Assembly

Using inline assembly for revert statements in Solidity can offer gas optimizations.
The typical `require` or `revert` statements in Solidity perform additional memory operations and type checks which can be avoided by using low-level assembly code.

In certain contracts, particularly those that might revert often or are otherwise sensitive to gas costs, using assembly to handle reversion can offer meaningful savings.
These savings primarily come from avoiding memory expansion costs and extra type checks that the Solidity compiler performs.

Example:
```solidity
/// calling restrictedAction(2) with a non-owner address: 24042
function restrictedAction(uint256 num)  external {
    require(owner == msg.sender, "caller is not owner");
    specialNumber = num;
}

// calling restrictedAction(2) with a non-owner address: 23734
function restrictedAction(uint256 num)  external {
    assembly {
        if sub(caller(), sload(owner.slot)) {
            mstore(0x00, 0x20) // store offset to where length of revert message is stored
            mstore(0x20, 0x13) // store length (19)
            mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
            revert(0x00, 0x60) // revert with data
        }
    }
    specialNumber = num;
}
```

<details>
<summary><i>12 issue instances in 8 files:</i></summary>

```solidity
File: src/UTB.sol

75: require(msg.value >= swapParams.amountIn, "not enough native");
```
[75](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L75)

```solidity
File: src/UTBFeeCollector.sol

29: require(signature.length == 65, "Invalid signature length");
54: require(recovered == signer, "Wrong signature");
```
[29](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L29) | [54](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L54)

```solidity
File: src/bridge_adapters/BaseAdapter.sol

12: require(
            msg.sender == address(bridgeExecutor),
            "Only bridge executor can call this"
        );
```
[12](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L12)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

91: require(
            destinationBridgeAdapter[dstChainId] != address(0),
            string.concat("dst chain address not set ")
        );
```
[91](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L91)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

157: require(
            dstAddr != address(0),
            string.concat("dst chain address not set ")
        );
```
[157](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L157)

```solidity
File: src/swappers/UniSwapper.sol

96: require(uniswap_router != address(0), "router not set");
```
[96](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L96)

```solidity
File: lib/decent-bridge/src/DcntEth.sol

9: require(msg.sender == router);
```
[9](https://github.com/decentxyz/decent-bridge/tree/main/src/DcntEth.sol#L9)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

38: require(gasCurrencyIsEth, "Gas currency is not ETH");
43: require(
            address(dcntEth) == msg.sender,
            "DecentEthRouter: only lz App can call"
        );
51: require(weth.balanceOf(address(this)) > amount, "not enough reserves");
62: require(balance >= amount, "not enough balance");
```
[38](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L38) | [43](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L43) | [51](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L51) | [62](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L62)
</details>


### [G-04] Use Cached Contracts for Multiple External Calls

When function makes multiple calls to the same external contract, it is more gas-efficient to use a local copy of the contract.
This is because the EVM will cache the contract in memory, and subsequent calls will be cheaper.
It's especially true for contracts that are large and/or have many functions.
```solidity
    // local cache -> 6561 gas
    IToken localCache = storageContract;
    localCache.externalCall();
    localCache.externalCall();

    // direct call 6683 gas
    storageContract.externalCall();
    storageContract.externalCall();
```

<details>
<summary><i>17 issue instances in 4 files:</i></summary>

```solidity
File: src/UTB.sol

/// @audit function performSwap() make external call of `wrapped` - 2 times
76: wrapped.deposit{value: swapParams.amountIn}();
98: wrapped.withdraw(amountOut);
/// @audit function _swapAndExecute() make external call of `executor` - 2 times
143: executor.execute{value: amountOut}(
153: executor.execute(
```
[76](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L76) | [98](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L98) | [143](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L143) | [153](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L153)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

/// @audit function estimateFees() make external call of `router` - 2 times
56: router.estimateSendAndCallFee(
57: router.MT_ETH_TRANSFER_WITH_PAYLOAD(),
```
[56](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L56) | [57](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L57)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

/// @audit function _bridgeWithPayload() make external call of `weth` - 2 times
178: weth.deposit{value: _amount}();
181: weth.transferFrom(msg.sender, address(this), _amount);
/// @audit function onOFTReceived() make external call of `weth` - 4 times
266: if (weth.balanceOf(address(this)) < _amount) {
273: weth.transfer(_to, _amount);
275: weth.withdraw(_amount);
279: weth.approve(address(executor), _amount);
```
[178](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L178) | [181](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L181) | [266](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L266) | [273](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L273) | [275](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L275) | [279](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L279)

```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

/// @audit function _executeWeth() make external call of `weth` - 5 times
30: uint256 balanceBefore = weth.balanceOf(address(this));
31: weth.approve(target, amount);
36: weth.transfer(from, amount);
41: (balanceBefore - weth.balanceOf(address(this)));
44: weth.transfer(from, remainingAfterCall);
```
[30](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L30) | [31](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L31) | [36](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L36) | [41](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L41) | [44](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L44)
</details>


### [G-05] Optimize Gas by Using Only Named Returns

The Solidity compiler can generate more efficient bytecode when using named returns.
It's recommended to replace anonymous returns with named returns for potential gas savings.

Example:
```solidity
/// 985 gas cost
function add(uint256 x, uint256 y) public pure returns (uint256) {
    return x + y;
}
/// 941 gas cost
function addNamed(uint256 x, uint256 y) public pure returns (uint256 res) {
    res = x + y;
}
```

<details>
<summary><i>14 issue instances in 4 files:</i></summary>

```solidity
File: src/UTB.sol

207: function approveAndCheckIfNative(
        BridgeInstructions memory instructions,
        uint256 amt2Bridge
    ) private returns (bool) {
259: function bridgeAndExecute(
        BridgeInstructions calldata instructions,
        FeeStructure calldata fees,
        bytes calldata signature
    )
        public
        payable
        retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
        returns (bytes memory)
    {
282: function callBridge(
        uint256 amt2Bridge,
        uint bridgeFee,
        BridgeInstructions memory instructions
    ) private returns (bytes memory) {
```
[207](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L207) | [259](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L259) | [282](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L282)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

29: function getId() public pure returns (uint8) {
66: function getBridgeToken(
        bytes calldata /*additionalArgs*/
    ) external view returns (address) {
72: function getBridgedAmount(
        uint256 amt2Bridge,
        address /*tokenIn*/,
        address /*tokenOut*/
    ) external pure returns (uint256) {
```
[29](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L29) | [66](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L66) | [72](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L72)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

40: function getId() public pure returns (uint8) {
60: function getBridgedAmount(
        uint256 amt2Bridge,
        address /*tokenIn*/,
        address /*tokenOut*/
    ) external pure returns (uint256) {
113: function getLzTxObj(
        bytes calldata additionalArgs
    ) private pure returns (IStargateRouter.lzTxObj memory) {
123: function getDstChainId(
        bytes calldata additionalArgs
    ) private pure returns (uint16) {
133: function getSrcPoolId(
        bytes calldata additionalArgs
    ) private pure returns (uint120) {
143: function getDstPoolId(
        bytes calldata additionalArgs
    ) private pure returns (uint120) {
```
[40](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L40) | [60](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L60) | [113](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L113) | [123](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L123) | [133](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L133) | [143](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L143)

```solidity
File: src/swappers/UniSwapper.sol

27: function getId() public pure returns (uint8) {
31: function updateSwapParams(
        SwapParams memory newSwapParams,
        bytes memory payload
    ) external pure returns (bytes memory) {
```
[27](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L27) | [31](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L31)
</details>


### [G-06] Use bytes32 in place of string

For strings of 32 char strings and below you can use bytes32 instead as it's more gas efficient

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

93: string.concat("dst chain address not set ")
```
[93](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L93)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

159: string.concat("dst chain address not set ")
```
[159](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L159)
</details>


### [G-07] Use Assembly for Hash Calculations

In certain cases, using inline assembly to calculate hashes can lead to significant gas savings. Solidity's built-in keccak256 function is convenient but costs more gas than the equivalent assembly code. However, it's important to note that using assembly should be done with care as it's less readable and could increase the risk of introducing errors.

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: src/UTBFeeCollector.sol

49: bytes32 constructedHash = keccak256(
50: abi.encodePacked(BANNER, keccak256(packedInfo))
```
[49](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L49) | [50](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L50)
</details>


### [G-08] Avoid Inverting `if`-`else` Conditions

Inverting the condition of an `if`-`else`-statement results in extra gas consumption.
Consider simplifying the condition to reduce gas costs.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

272: if (!gasCurrencyIsEth || !deliverEth) {
                weth.transfer(_to, _amount);
            } else {
                weth.withdraw(_amount);
                payable(_to).transfer(_amount);
            }
```
[272](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L272)

```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

76: if (!gasCurrencyIsEth || !deliverEth) {
            _executeWeth(from, target, amount, callPayload);
        } else {
            _executeEth(from, target, amount, callPayload);
        }
```
[76](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L76)
</details>


### [G-09] Optimize External Calls with Assembly for Memory Efficiency

Using interfaces to make external contract calls in Solidity is convenient but can be inefficient in terms of memory utilization.
Each such call involves creating a new memory location to store the data being passed, thus incurring memory expansion costs. 

Inline assembly allows for optimized memory usage by re-using already allocated memory spaces or using the scratch space for smaller datasets.
This can result in notable gas savings, especially for contracts that make frequent external calls.

Additionally, using inline assembly enables important safety checks like verifying if the target address has code deployed to it using `extcodesize(addr)` before making the call, mitigating risks associated with contract interactions.

<details>
<summary><i>14 issue instances in 5 files:</i></summary>

```solidity
File: src/UTB.sol

152: IERC20(tokenOut).approve(address(executor), amountOut);
216: IERC20(bridgeToken).approve(address(bridgeAdapter), amt2Bridge);
```
[152](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L152) | [216](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L216)

```solidity
File: src/UTBExecutor.sol

61: IERC20(token).transferFrom(msg.sender, address(this), amount);
62: IERC20(token).approve(paymentOperator, amount);
73: uint remainingBalance = IERC20(token).balanceOf(address(this)) -
            initBalance;
80: IERC20(token).transfer(refund, remainingBalance);
```
[61](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L61) | [62](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L62) | [73](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L73) | [80](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L80)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

109: IERC20(bridgeToken).transferFrom(
                msg.sender,
                address(this),
                amt2Bridge
            );
114: IERC20(bridgeToken).approve(address(router), amt2Bridge);
```
[109](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L109) | [114](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L114)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

88: IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);
89: IERC20(bridgeToken).approve(address(router), amt2Bridge);
```
[88](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88) | [89](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L89)

```solidity
File: src/swappers/UniSwapper.sol

44: IERC20(token).transfer(user, amount);
55: IERC20(token).transfer(recipient, amount);
138: amountOut = IV3SwapRouter(uniswap_router).exactInput(params);
159: amountIn = IV3SwapRouter(uniswap_router).exactOutput(params);
```
[44](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L44) | [55](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L55) | [138](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L138) | [159](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L159)
</details>


### [G-10] Unlimited gas consumption risk due to external call recipients

When calling an external function without specifying a gas limit , the called contract may consume all the remaining gas, causing the tx to be reverted.
To mitigate this, it is recommended to explicitly set a gas limit when making low level external calls.

<details>
<summary><i>5 issue instances in 2 files:</i></summary>

```solidity
File: src/UTBExecutor.sol

52: (success, ) = target.call{value: amount}(payload);
65: (success, ) = target.call{value: extraNative}(payload);
70: (success, ) = target.call(payload);
```
[52](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L52) | [65](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L65) | [70](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L70)

```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

33: (bool success, ) = target.call(callPayload);
61: (bool success, ) = target.call{value: amount}(callPayload);
```
[33](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L33) | [61](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L61)
</details>


### [G-11] Reduce deployment costs by tweaking contracts' metadata

The Solidity compiler appends 53 bytes of metadata to the smart contract code which translates to an extra 10,600 gas (200 per bytecode) + the calldata cost (16 gas per non-zero bytes, 4 gas per zero-byte).
This translates to up to 848 additional gas in calldata cost.
One way to reduce this cost is by optimizing the IPFS hash that gets appended to the smart contract code.

Why is this important?
- The metadata adds an extra 53 bytes, resulting in an additional 10,600 gas cost for deployment.
- It also incurs up to 848 additional gas in calldata cost.

Options to Reduce Gas:
1. Use the `--no-cbor-metadata` compiler option to exclude metadata, but this might affect contract verification.
2. Mine for code comments that lead to an IPFS hash with more zeros, reducing calldata costs.

<details>
<summary><i>11 issue instances in 11 files:</i></summary>

```solidity
File: src/UTB.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L1)

```solidity
File: src/UTBExecutor.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L1)

```solidity
File: src/UTBFeeCollector.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L1)

```solidity
File: src/bridge_adapters/BaseAdapter.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L1)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L1)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L1)

```solidity
File: src/swappers/SwapParams.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L1)

```solidity
File: src/swappers/UniSwapper.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L1)

```solidity
File: lib/decent-bridge/src/DcntEth.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/decentxyz/decent-bridge/tree/main/src/DcntEth.sol#L1)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L1)

```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L1)
</details>


### [G-12] Unused `private` functions should be removed to save deployment gas

All private functions that are never used can be safely removed or commented out to save gas.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

100: function getValue(
        bytes calldata additionalArgs,
        uint256 amt2Bridge
    ) private view returns (uint value)
```
[100](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L100)
</details>


### [G-13] Use `revert()` to gain maximum gas savings

If you dont need Error messages, or you want gain maximum gas savings - `revert()` is a cheapest way to revert transaction in terms of gas.
```solidity
    revert(); // 117 gas 
    require(false); // 132 gas
    revert CustomError(); // 157 gas
    assert(false); // 164 gas
    revert("Custom Error"); // 406 gas
    require(false, "Custom Error"); // 421 gas
```


<details>
<summary><i>12 issue instances in 8 files:</i></summary>

```solidity
File: src/UTB.sol

75: require(msg.value >= swapParams.amountIn, "not enough native")
```
[75](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L75)

```solidity
File: src/UTBFeeCollector.sol

29: require(signature.length == 65, "Invalid signature length")
54: require(recovered == signer, "Wrong signature")
```
[29](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L29) | [54](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L54)

```solidity
File: src/bridge_adapters/BaseAdapter.sol

12: require(
            msg.sender == address(bridgeExecutor),
            "Only bridge executor can call this"
        )
```
[12](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L12)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

91: require(
            destinationBridgeAdapter[dstChainId] != address(0),
            string.concat("dst chain address not set ")
        )
```
[91](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L91)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

157: require(
            dstAddr != address(0),
            string.concat("dst chain address not set ")
        )
```
[157](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L157)

```solidity
File: src/swappers/UniSwapper.sol

96: require(uniswap_router != address(0), "router not set")
```
[96](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L96)

```solidity
File: lib/decent-bridge/src/DcntEth.sol

9: require(msg.sender == router)
```
[9](https://github.com/decentxyz/decent-bridge/tree/main/src/DcntEth.sol#L9)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

38: require(gasCurrencyIsEth, "Gas currency is not ETH")
43: require(
            address(dcntEth) == msg.sender,
            "DecentEthRouter: only lz App can call"
        )
51: require(weth.balanceOf(address(this)) > amount, "not enough reserves")
62: require(balance >= amount, "not enough balance")
```
[38](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L38) | [43](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L43) | [51](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L51) | [62](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L62)
</details>


### [G-14] Optimize Ether Transfers with `receive()` Function

Consider using `receive()` function instead of a specific `deposit()` (or similar) function.
If there are several functions in the contract that can receive Ether, it is recommended to use `receive()` for the most frequently used function.
```solidity
function deposit() external payable { // 5401 gas
    // your logic
}

receive() external payable {  // 5356 gas
    // your logic
}
```

The `receive()` or `fallback()` function can handle incoming Ether transfers directly, providing more gas-efficient way to manage deposits.

<details>
<summary><i>13 issue instances in 7 files:</i></summary>

```solidity
File: src/UTB.sol

108: function swapAndExecute(
        SwapAndExecuteInstructions calldata instructions,
        FeeStructure calldata fees,
        bytes calldata signature
    )
        public
        payable
        retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
259: function bridgeAndExecute(
        BridgeInstructions calldata instructions,
        FeeStructure calldata fees,
        bytes calldata signature
    )
        public
        payable
        retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
        returns (bytes memory)
```
[108](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L108) | [259](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L259)

```solidity
File: src/UTBExecutor.sol

19: function execute(
        address target,
        address paymentOperator,
        bytes memory payload,
        address token,
        uint amount,
        address payable refund
    ) public payable onlyOwner
```
[19](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L19)

```solidity
File: src/UTBFeeCollector.sol

44: function collectFees(
        FeeStructure calldata fees,
        bytes memory packedInfo,
        bytes memory signature
    ) public payable onlyUtb
```
[44](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L44)

```solidity
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
    ) public payable onlyUtb returns (bytes memory bridgePayload)
```
[81](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L81)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

69: function bridge(
        uint256 amt2Bridge,
        SwapInstructions memory postBridge,
        uint256 dstChainId,
        address target,
        address paymentOperator,
        bytes memory payload,
        bytes calldata additionalArgs,
        address payable refund
    ) public payable onlyUtb returns (bytes memory bridgePayload)
```
[69](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L69)

```solidity
File: src/swappers/UniSwapper.sol

100: function swapNoPath(
        SwapParams memory swapParams,
        address receiver,
        address refund
    ) public payable returns (address tokenOut, uint256 amountOut)
123: function swapExactIn(
        SwapParams memory swapParams, // SwapParams is a struct
        address receiver
    ) public payable routerIsSet returns (uint256 amountOut)
143: function swapExactOut(
        SwapParams memory swapParams,
        address receiver,
        address refundAddress
    ) public payable routerIsSet returns (uint256 amountIn)
```
[100](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L100) | [123](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L123) | [143](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L143)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

197: function bridgeWithPayload(
        uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        bool deliverEth,
        uint64 _dstGasForCall,
        bytes memory additionalPayload
    ) public payable
218: function bridge(
        uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        uint64 _dstGasForCall,
        bool deliverEth // if false, delivers WETH
    ) public payable
302: function addLiquidityEth()
        public
        payable
        onlyEthChain
        userDepositing(msg.value)
322: function addLiquidityWeth(
        uint256 amount
    ) public payable userDepositing(amount)
```
[197](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L197) | [218](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L218) | [302](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L302) | [322](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L322)
</details>


### [G-15] Avoid contract existence checks by using low-level calls

Before version 0.8.10, the Solidity compiler would insert extra code, such as EXTCODESIZE (costing 100 gas), to check the existence of a contract for external function calls.
Newer versions, starting from 0.8.10, no longer insert these checks if the external call has a return value.
You can achieve similar behavior in earlier Solidity versions by using low-level calls like `call`.
This low-level call don't check for contract existence, saving gas costs.

<details>
<summary><i>47 issue instances in 7 files:</i></summary>

```solidity
File: src/UTB.sol

78: swapInstructions.swapPayload = swapper.updateSwapParams(
83: IERC20(swapParams.tokenIn).transferFrom(
90: IERC20(swapParams.tokenIn).approve(
95: (tokenOut, amountOut) = swapper.swap(swapInstructions.swapPayload);
98: wrapped.withdraw(amountOut);
152: IERC20(tokenOut).approve(address(executor), amountOut);
153: executor.execute(
188: ).getBridgedAmount(amountOut, tokenOut, newPostSwapParams.tokenIn);
194: ]).updateSwapParams(
212: address bridgeToken = bridgeAdapter.getBridgeToken(
216: IERC20(bridgeToken).approve(address(bridgeAdapter), amt2Bridge);
236: IERC20(fees.feeToken).transferFrom(
241: IERC20(fees.feeToken).approve(
327: swappers[s.getId()] = swapper;
336: bridgeAdapters[b.getId()] = bridge;
```
[78](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L78) | [83](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L83) | [90](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L90) | [95](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L95) | [98](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L98) | [152](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L152) | [153](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L153) | [188](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L188) | [194](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L194) | [212](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L212) | [216](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L216) | [236](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L236) | [241](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L241) | [327](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L327) | [336](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L336)

```solidity
File: src/UTBExecutor.sol

61: IERC20(token).transferFrom(msg.sender, address(this), amount);
62: IERC20(token).approve(paymentOperator, amount);
80: IERC20(token).transfer(refund, remainingBalance);
```
[61](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L61) | [62](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L62) | [80](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L80)

```solidity
File: src/UTBFeeCollector.sol

56: IERC20(fees.feeToken).transferFrom(
71: payable(owner).transfer(amount);
73: IERC20(token).transfer(owner, amount);
```
[56](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L56) | [71](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L71) | [73](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L73)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

56: router.estimateSendAndCallFee(
57: router.MT_ETH_TRANSFER_WITH_PAYLOAD(),
109: IERC20(bridgeToken).transferFrom(
114: IERC20(bridgeToken).approve(address(router), amt2Bridge);
139: IERC20(swapParams.tokenIn).transferFrom(
145: IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn);
147: IUTB(utb).receiveFromBridge(
```
[56](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L56) | [57](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L57) | [109](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L109) | [114](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L114) | [139](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L139) | [145](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L145) | [147](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L147)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

88: IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);
89: IERC20(bridgeToken).approve(address(router), amt2Bridge);
207: IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn);
209: IUTB(utb).receiveFromBridge(
```
[88](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88) | [89](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L89) | [207](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L207) | [209](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L209)

```solidity
File: src/swappers/UniSwapper.sol

44: IERC20(token).transfer(user, amount);
55: IERC20(token).transfer(recipient, amount);
83: IERC20(swapParams.tokenIn).transferFrom(
130: .ExactInputParams({
137: IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
138: amountOut = IV3SwapRouter(uniswap_router).exactInput(params);
150: .ExactOutputParams({
158: IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
159: amountIn = IV3SwapRouter(uniswap_router).exactOutput(params);
```
[44](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L44) | [55](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L55) | [83](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L83) | [130](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L130) | [137](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L137) | [138](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L138) | [150](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L150) | [158](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L158) | [159](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L159)

```solidity
File: lib/decent-bridge/src/DecentBridgeExecutor.sol

31: weth.approve(target, amount);
36: weth.transfer(from, amount);
44: weth.transfer(from, remainingAfterCall);
60: weth.withdraw(amount);
63: payable(from).transfer(amount);
75: weth.transferFrom(msg.sender, address(this), amount);
```
[31](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L31) | [36](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L36) | [44](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L44) | [60](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L60) | [63](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L63) | [75](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentBridgeExecutor.sol#L75)
</details>


### [G-16] Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead. The Ethereum Virtual Machine (EVM) operates on 32 bytes at a time. Therefore, if an element is smaller than 32 bytes, the EVM must use more operations to reduce the size of the element from 32 bytes to the desired size. 

Operations involving smaller size uints/ints cost extra gas due to the compiler having to clear the higher bits of the memory word before operating on the small size integer. This also includes the associated stack operations of doing so. 

It's recommended to use larger sizes and downcast where needed to optimize for gas efficiency.

<details>
<summary><i>31 issue instances in 6 files:</i></summary>

```solidity
File: src/UTB.sol

21: mapping(uint8 => address) public swappers;
22: mapping(uint8 => address) public bridgeAdapters;
```
[21](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L21) | [22](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L22)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

13: uint8 public constant BRIDGE_ID = 0;
17: mapping(uint16 => uint256) chainIdLookup;
30: function getId() public pure returns (uint8) {
96: uint64 dstGas = abi.decode(additionalArgs, (uint64));
```
[13](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L13) | [17](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L17) | [30](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L30) | [96](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L96)

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

22: uint8 public constant BRIDGE_ID = 1;
23: uint8 public constant SG_FEE_BPS = 6;
27: mapping(uint16 => uint256) chainIdLookup;
41: function getId() public pure returns (uint8) {
126: ) private pure returns (uint16) {
136: ) private pure returns (uint120) {
146: ) private pure returns (uint120) {
184: uint16, // _srcChainid
        bytes memory, // _srcAddress
        uint256, // _nonce
        address, // _token
        uint256, // amountLD
        bytes memory payload
    ) external override onlyExecutor {
```
[22](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L22) | [23](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L23) | [27](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L27) | [41](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L41) | [126](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L126) | [136](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L136) | [146](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L146) | [184](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L184)

```solidity
File: src/swappers/SwapParams.sol

5: uint8 constant EXACT_IN = 0;
6: uint8 constant EXACT_OUT = 1;
```
[5](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L5) | [6](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/SwapParams.sol#L6)

```solidity
File: src/swappers/UniSwapper.sol

16: uint8 public constant SWAPPER_ID = 0;
28: function getId() public pure returns (uint8) {
```
[16](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L16) | [28](https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L28)

```solidity
File: lib/decent-bridge/src/DecentEthRouter.sol

17: uint8 public constant MT_ETH_TRANSFER = 0;
18: uint8 public constant MT_ETH_TRANSFER_WITH_PAYLOAD = 1;
20: uint16 public constant PT_SEND_AND_CALL = 1;
24: mapping(uint16 => address) public destinationBridges;
74: uint16 _dstChainId,
        address _routerAddress
    ) public onlyOwner {
81: uint8 msgType,
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
114: uint8 msgType,
        uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        uint64 _dstGasForCall,
        bool deliverEth,
        bytes memory payload
    ) public view returns (uint nativeFee, uint zroFee) {
149: uint8 msgType,
        uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        uint64 _dstGasForCall,
        bytes memory additionalPayload,
        bool deliverEth
    ) internal {
198: uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        bool deliverEth,
        uint64 _dstGasForCall,
        bytes memory additionalPayload
    ) public payable {
219: uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        uint64 _dstGasForCall,
        bool deliverEth // if false, delivers WETH
    ) public payable {
238: uint16 _srcChainId,
        bytes calldata,
        uint64,
        bytes32,
        uint _amount,
        bytes memory _payload
    ) external override onlyLzApp {
245: (uint8 msgType, address _from, address _to, bool deliverEth) = abi
            .decode(_payload, (uint8, address, address, bool));
253: (uint8, address, address, bool, bytes)
            );
```
[17](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L17) | [18](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L18) | [20](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L20) | [24](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L24) | [74](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L74) | [81](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L81) | [114](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L114) | [149](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L149) | [198](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L198) | [219](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L219) | [238](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L238) | [245](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L245) | [253](https://github.com/decentxyz/decent-bridge/tree/main/src/DecentEthRouter.sol#L253)
</details>
