## Low Issues

### [L-1] `UTB::registerSwapper()` and `UTB::registerBridge()` do not validate input's existence

`UTB::registerSwapper()` and `UTB::registerBridge()` lets owners of the `UTB` contract overwrite existing data in the mappings without performing any checks:

```solidity
/**
  * @dev Registers and maps a swapper to a swapper ID.
  * @param swapper The address of the swapper.
  */
function registerSwapper(address swapper) public onlyOwner {
    ISwapper s = ISwapper(swapper);
    swappers[s.getId()] = swapper;
}

/**
  * @dev Registers and maps a bridge adapter to a bridge adapter ID.
  * @param bridge The address of the bridge adapter.
  */
function registerBridge(address bridge) public onlyOwner {
    IBridgeAdapter b = IBridgeAdapter(bridge);
    bridgeAdapters[b.getId()] = bridge;
}
```

In my opinion, calls should check if the swappers and bridge adapters already exist and if they do, revert. An optional second param can be used to mandate to the function whether it should perform a replacement or new creation. This could help prevent `bridgeAdapters` and `swappers` accidentally being overwritten with wrong ones.

### [L-2] Lack of specifiable gas limit some external calls

Users can't control gas limit for external calls. This might lead to their gas being consumed unexpectedly.

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L337-L339

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L61

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L52-L54

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L65-L70

### [L-3] 1 wei will always be stuck inside `DecentEthRouter` due to off-by-one error

```solidity
modifier onlyIfWeHaveEnoughReserves(uint256 amount) {
    require(weth.balanceOf(address(this)) > amount, "not enough reserves");
    _;
}

/// @inheritdoc IDecentEthRouter
function redeemEth(
    uint256 amount
) public onlyIfWeHaveEnoughReserves(amount) {
    dcntEth.transferFrom(msg.sender, address(this), amount);
    weth.withdraw(amount);
    payable(msg.sender).transfer(amount);
}

/// @inheritdoc IDecentEthRouter
function redeemWeth(
    uint256 amount
) public onlyIfWeHaveEnoughReserves(amount) {
    dcntEth.transferFrom(msg.sender, address(this), amount);
    weth.transfer(msg.sender, amount);
}
```

Due to the usage of `>` instead of `>=`, 1 extra wei will always be needed in this contract to peform the `redeemEth()` and `redeemWeth()` actions.

To mitigate, the require statement inside the modifier should be changed to:

```solidity
require(weth.balanceOf(address(this)) >= amount, "not enough reserves");
```

### [L-4] `DecentBridgeAdapter::estimateFees()` and `DecentBridgeAdapter::bridge()` does not check if `dstChainId` is valid.

```solidity
function estimateFees(
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
    router.estimateSendAndCallFee(
        router.MT_ETH_TRANSFER_WITH_PAYLOAD(),
        lzIdLookup[dstChainId],
        target,
        swapParams.amountIn,
        dstGas,
        false,
        payload
    );
}
```

Passing in an unsupported `dstChainId` will lead to the call reverting and the caller having their gas lost unexpectedly. Consider checking the if the `dstChainId` is already in the `lzIdLookup` mapping before proceeding with the `router` call.

### [L-5] `UTB::performSwap()` does not strictly impose that `msg.value == swapParams.amountIn`, leading to potential loss of funds

```solidity
if (swapParams.tokenIn == address(0)) {
    require(msg.value >= swapParams.amountIn, "not enough native");
    wrapped.deposit{value: swapParams.amountIn}();
    swapParams.tokenIn = address(wrapped);
    swapInstructions.swapPayload = swapper.updateSwapParams(
        swapParams,
        swapInstructions.swapPayload
    );
}
```

The code uses:

```solidity
require(msg.value >= swapParams.amountIn, "not enough native");
```

It should be changed to:

```solidity
require(msg.value == swapParams.amountIn, "not enough native");
```

This will help prevent users accidentally sending more funds than needed and having them stuck in the contract forever.

## Informational issues

### [I-1] Use string literals instead of `string.concat()`

Pieces of code in `DecentBridgeAdapter` and `StargateBridgeAdapter` use `string.concat()` where simple string literals can be used:

`DecentBridgeAdapter`:

```solidity
require(
    destinationBridgeAdapter[dstChainId] != address(0),
    string.concat("dst chain address not set ")
);
```

`StargateBridgeAdapter`:

```solidity
require(
    dstAddr != address(0),
    string.concat("dst chain address not set ")
);
```

### [I-2] `UniSwapper::SWAPPER_ID`, `DecentBridgeAdapter::BRIDGE_ID`, and `StargateBridgeAdapter::BRIDGE_ID` are `constant`, should be `immutable`

`UniSwapper::SWAPPER_ID` is hardcoded as a constant:

```solidity
uint8 public constant SWAPPER_ID = 0;
```

I would suggest changing it into `immutable` and setting the id in the constructor, so all `UniSwapper` instances can use the same code rather than having to go in and change it manually for each new one.

Same goes for `DecentBridgeAdapter::BRIDGE_ID` and `StargateBridgeAdapter::BRIDGE_ID`.

### [I-3] Unused function `StargateBridgeAdapter::getValue()`

The function `StargateBridgeAdapter::getValue()` isn't used anywhere. Double check if something was missed. If it's indeed not needed, remove it.
