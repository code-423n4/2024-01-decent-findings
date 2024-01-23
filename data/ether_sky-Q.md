[L-1] There is no check between `gasIsEth` and `bridgeToken`.

There is a strict relationship between `gasIsEth` and `bridgeToken`.
https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/DecentBridgeAdapter.sol#L21-L24
```
constructor(bool _gasIsEth, address _bridgeToken) BaseAdapter() {
    gasIsEth = _gasIsEth;
    bridgeToken = _bridgeToken;
}
```
If the `bridge token` is set, the user should transfer this `bridge token`. 
If not, the user should send `ETH`.
https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTB.sol#L216
```
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
In the `Bridge Adapter`, we assume that the `adapter` receives the `bridge token` when `gasIsEth` is `false` and receives `ETH` when `gasIsEth` is `true`.
https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/DecentBridgeAdapter.sol#L108-L115
```
function bridge() {
    if (!gasIsEth) {
        IERC20(bridgeToken).transferFrom(
            msg.sender,
            address(this),
            amt2Bridge
        );
        IERC20(bridgeToken).approve(address(router), amt2Bridge);
    }
}
```
Therefore, the following relationship should be satisfied.
`bridgeToken = weth if gasIsEth is false, 0 if gasIsEth is true`

And `gasIsEth` should be the same as `gasCurrencyIsEth` in `DecentEthRouter`.
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L22
```
contract DecentEthRouter is IDecentEthRouter, IOFTReceiverV2, Owned {
    bool public gasCurrencyIsEth;
}
```

[L-2] The transaction can fail when there is not enough `DcntEth` tokens in the `DecentEthRouter` contract.

When users attempt to swap tokens, they send `WETH` tokens to the `router`. 
The `router` receives `WETH` tokens and removes an equivalent amount of `DcntEth` tokens.
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L177-L193
```
function _bridgeWithPayload (
    if (gasCurrencyIsEth) {
        weth.deposit{value: _amount}();
        gasValue = msg.value - _amount;
    } else {
        weth.transferFrom(msg.sender, address(this), _amount);
        gasValue = msg.value;
    }

    dcntEth.sendAndCall{value: gasValue}(
        address(this), // from address that has dcntEth (so DecentRouter)
        _dstChainId,
        destinationBridge, // toAddress
        _amount, // amount
        payload, //payload (will have recipients address)
        _dstGasForCall, // dstGasForCall
        callParams // refundAddress, zroPaymentAddress, adapterParams
    );
}
```
So, if the `router` does not have enough `DcntEth` tokens, the transaction can fail. 
While anyone can deposit `DcntEth` to the `router` or the `owner` can mint tokens to the `router`, for safety, we should add the following.
```
function bridge() {
+    if (dcntEth.balanceOf(address(this)) < _amount) {
+        dcntEth.mint(address(this), _amount);
+    }
}
```
This is possible because the `router` has the ability to mint `DcntEth` tokens.

[L-3] Users can use the protocol without paying fees.

When users attempt to swap tokens using `swapAndExecute` or `bridgeAndExecute`, they should pay protocol fees.
https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTB.sol#L115
```
function swapAndExecute(
    SwapAndExecuteInstructions calldata instructions,
    FeeStructure calldata fees,
    bytes calldata signature
)
    public
    payable
    retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
{
}
```

However, users can also swap tokens using the `receiveFromBridge` function, and they are not required to pay protocol fees.
https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTB.sol#L311-L319
```
function receiveFromBridge(
    SwapInstructions memory postBridge,
    address target,
    address paymentOperator,
    bytes memory payload,
    address payable refund
) public {
    _swapAndExecute(postBridge, target, paymentOperator, payload, refund);
}
```

[L-4] Missing `payable`

The `execute` function in `UTBExecuto`r is not marked as `payable`.
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBExecutor.sol#L49
```
function execute(
    address target,
    address paymentOperator,
    bytes memory payload,
    address token,
    uint amount,
    address payable refund,
    uint extraNative
) public onlyOwner {
}
```

[L-5] Unable to invoke the `receiveFromBridge` function from `DecentBridgeExecutor` properly.

If `gasCurrencyIsEth` and `deliverEth` are both `true` in the `execute` function, the `_executeEth` function is called. 
In this function, we perform a withdrawal of `WETH` and send `ETH` to the `Bridge Adapter`.
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L61
```
function _executeEth() {
    weth.withdraw(amount);
    (bool success, ) = target.call{value: amount}(callPayload);
}
```

However, in the `receiveFromBridge` function, we only accept `ERC20` tokens.
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/bridge_adapters/DecentBridgeAdapter.sol#L139-L143
```
function receiveFromBridge() {
    IERC20(swapParams.tokenIn).transferFrom(
        msg.sender,
        address(this),
        swapParams.amountIn
    );
}
```

[L-5] There's a possibility that some funds may become trapped in the `Bridge Adapter`.

In the `_bridgeWithPayload` function, we set the `Bridge Adapter` as the `refund` recipient.
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L171
```
function _bridgeWithPayload() {
    ICommonOFT.LzCallParams memory callParams = ICommonOFT.LzCallParams({
        refundAddress: payable(msg.sender),
        zroPaymentAddress: address(0x0),
        adapterParams: adapterParams
    });
}
```
Consequently, certain funds will be accumulated as fees within the `Bridge Adapter`. 
There should be no remaining funds left in the Bridge Adapter in any case.



