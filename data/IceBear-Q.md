## [L-01] Potential DoS  in collectFees.
Missing checks for address(0) issue can be found in automated findings, but here it could lead to a DoS, so two scenarios are described:
1. Owner hasn't set a signer, and the signer is defaulted to the zero address.
2. Owner sets the signer to the zero address.

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L18
```
/**
     * @dev Sets the signer used for fee verification.
     * @param _signer The address of the signer.
     */
    function setSigner(address _signer) public onlyOwner {
        signer = _signer;
    }
```


This would fail to pass through splitSignature and render collectFees() unusable.

## [L-02] setRouter lack access control

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L20
```
 function setRouter(address _router) public {
        router = _router;
    }
```
setRouter lack access control
Anyone can set themselves as the router and pass through onlyRouter().
Can mint or burn tokens arbitrarily.

## [L-03] Do not hardcode useZero field to false
Do not hardcode useZro to false when estimating fees, pass it as a parameter instead. 
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L142
```
dcntEth.estimateSendAndCallFee(
                _dstChainId,
                destinationBridge,
                _amount,
                _payload,
                _dstGasForCall,
                false, // useZero
                adapterParams // Relayer adapter parameters
                // contains packet type (send and call in this case) and gasAmount
            );
```
See point 6 in this [integration checklist](https://layerzero.gitbook.io/docs/troubleshooting/layerzero-integration-checklist) provided by the LayerZero team.

## [L-04] Hardcoded GAS_FOR_RELAY can be set higher.
Layer Zero acknowledges that a cross-chain call can use [more or less gas than the standard 200k](https://layerzero.gitbook.io/docs/evm-guides/advanced/relayer-adapter-parameters). 
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L96
```
uint256 GAS_FOR_RELAY = 100000;
        uint256 gasAmount = GAS_FOR_RELAY + _dstGasForCall;
```
Adjust to 200,000 gas.

## [L-05] Use quoteLayerZeroFee to calculate the fee
Ensure that callBridge in StargateBridgeAdapter can execute as expected.
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L170
```
    function callBridge(
        uint256 amt2Bridge,
        uint256 dstChainId,
        bytes memory bridgePayload,
        bytes calldata additionalArgs,
        address payable refund
    ) private {
        router.swap{value: msg.value}(
            getDstChainId(additionalArgs), //lzBridgeData._dstChainId, // send to LayerZero chainId
            getSrcPoolId(additionalArgs), //lzBridgeData._srcPoolId, // source pool id
            getDstPoolId(additionalArgs), //lzBridgeData._dstPoolId, // dst pool id
            refund, // refund adddress. extra gas (if any) is returned to this address
            amt2Bridge, // quantity to swap
            (amt2Bridge * (10000 - SG_FEE_BPS)) / 10000, // the min qty you would accept on the destination, fee is 6 bips
            getLzTxObj(additionalArgs), // additional gasLimit increase, airdrop, at address
            abi.encodePacked(getDestAdapter(dstChainId)),
            bridgePayload // bytes param, if you wish to send additional payload you can abi.encode() them here
        );
    }
```
Currently using msg.value in router.swap, to ensure that bridge() can be executed as expected.
[Use quoteLayerZeroFee()](https://stargateprotocol.gitbook.io/stargate/developers/cross-chain-swap-fee) to get the fee required to call swap(). The fee ensures the cross chain message is paid for. Use it as the { value: xxxx } passed to the actual swap() method when you perform the swap.