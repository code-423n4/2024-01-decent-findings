### Low Risk

| Count | Title |
| --- | --- |
| [L-01](#l-01-zropaymentaddress-should-not-be-hardcoded-to-address0) | zroPaymentAddress should not be hardcoded to address(0) |
| [L-02](#l-02-do-not-hardcode-_usezro-to-false-when-estimating-layerzero-fees) | Do not hardcode _useZro to false when estimating layerZero fees |
| [L-03](#l-03-off-by-one-will-prevent-both-weth-and-eth-full-redeems) | Off-by one will prevent full redeems in DecentEthRouter |

| Total Low Risk Issues | 3 |
| --- | --- |

## Low Risks

## [L-01] `zroPaymentAddress` should not be hardcoded to `address(0)`

**Issue Description:** In `DecentEthRouter::_bridgeWithPayload`, `zroPaymentAddress` is hardcoded to `address(0)` which is wrong according to the LayerZero integration checklist.

> *"Do not hardcode address zero `address(0)` as zroPaymentAddress when estimating fees and sending messages. Pass it as a parameter instead."*
> 

```solidity
ICommonOFT.LzCallParams memory callParams = ICommonOFT.LzCallParams({
    refundAddress: payable(msg.sender),
    zroPaymentAddress: address(0x0), //@audit should not be hardcoded
    adapterParams: adapterParams
});
```

**Recommendation:** Pass `zroPaymentAddress` as parameter

## [L-02] Do not hardcode `_useZro` to false when estimating layerZero fees

**Issue Description:** In `DecentEthRouter::estimateSendAndCallFee`, `_useZro` is hardcoded to `false` which is wrong according to the LayerZero integration checklist.

> *"Do not hardcode `useZro` to `false` when estimating fees and sending messages. Pass it as a parameter instead."*
> 

```solidity
function estimateSendAndCallFee(
    uint8 msgType,
    uint16 _dstChainId,
    address _toAddress,
    uint _amount,
    uint64 _dstGasForCall,
    bool deliverEth,
    bytes memory payload
) public view returns (uint nativeFee, uint zroFee) {
    (
        bytes32 destinationBridge,
        bytes memory adapterParams,
        bytes memory _payload
    ) = _getCallParams(
            msgType,
            _toAddress,
            _dstChainId,
            _dstGasForCall,
            deliverEth,
            payload
        );

    return
        dcntEth.estimateSendAndCallFee(
            _dstChainId,
            destinationBridge,
            _amount,
            _payload,
            _dstGasForCall,
            false, // useZero // @audit should not be hardcoded
            adapterParams // Relayer adapter parameters
            // contains packet type (send and call in this case) and gasAmount
        );
}
```

**Recommendation:** Pass `_useZro` as parameter

## [L-03] Off-by one will prevent both `WETH` and `ETH` full redeems.

**Issue Description:** 

This functions will be used to when there is not enough liquidity when the bridge message reaches the destination chain user will be refunded in form of dcntETH token and he will have to redeem the in either one of the functions: 

- `redeemEth`
- `redeemWeth`

Both have `onlyIfWeHaveEnoughReserves` modifier applied:

```solidity
modifier onlyIfWeHaveEnoughReserves(uint256 amount) {
        require(weth.balanceOf(address(this)) > amount, "not enough reserves");
        _;
    }
```

Since this contract will not hold any excess `WETH` and `ETH`, user who redeems will always lose 1 wei. 

**Recommendation:** 

**Modify the** `onlyIfWeHaveEnoughReserves`:

```diff
modifier onlyIfWeHaveEnoughReserves(uint256 amount) {
-        require(weth.balanceOf(address(this)) > amount, "not enough reserves");
+        require(weth.balanceOf(address(this)) >= amount, "not enough reserves");
        _;
    }
```