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