## L-01: forceResumeReceive() not implemented
The protocol doesn't implement a very important [LayerZero](https://layerzero.gitbook.io/docs/layerzero-tooling/best-practice) function.

It is highly recommended by LayerZero to implement ILayerZeroUserApplicationConfig.forceResumeReceive(), because;
> Implementing this interface will provide you with forceResumeReceive which in the worse case can allow the owner/multisig to unblock the queue of messages if something unexpected happens.

### Mitigation
Consider implementing [ILayerZeroUserApplicationConfig.forceResumeReceive()](https://layerzero.gitbook.io/docs/evm-guides/evm-solidity-interfaces/ilayerzerouserapplicationconfig):
```solidity
    // @notice Only when the UA needs to resume the message flow in blocking mode and clear the stored payload
    // @param _srcChainId - the chainId of the source chain
    // @param _srcAddress - the contract address of the source contract at the source chain
    function forceResumeReceive(uint16 _srcChainId, bytes calldata _srcAddress) external;
```

## L-02: `zroPaymentAddress` hardcoded as zero address
[LayerZero](https://layerzero.gitbook.io/docs/troubleshooting/layerzero-integration-checklist) highly discoraged hardcording [zroPaymentAddress](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L172) as address(0): Do not hardcode address zero (address(0)) as zroPaymentAddress when estimating fees and sending messages. 

### Mitigation
Cosider paasing it as a parameter instead:
```diff
        ICommonOFT.LzCallParams memory callParams = ICommonOFT.LzCallParams({
            refundAddress: payable(msg.sender),
-           zroPaymentAddress: address(0x0),
+           zroPaymentAddress: _zroPaymentAddress,
            adapterParams: adapterParams
        });
```

## L-03: 0.8.20 breaks on Optimism & Base chains
Project may fail to be deployed to chains not compatible with Shanghai hardfork. The Decent protocol targets to be deployed on all layerZero supporting chains including Optimism & Base. All of the contracts are using using Solidity 0.8.20 compiler version (defined in foundry.toml). 0.8.20 version of the compiler uses the new PUSH0 opcode introduced in the Shanghai hard fork, which is now the default EVM version in the compiler and the one being currently used to compile the project.

### Mitigation
Consider Downgrading Solidity version to 0.8.19 for safe deployments on all chains.

## L-04: Lack of Deadline for Uniswap Swap functions
Uniswapper.swapExactIn() & Uniswapper.swapExactOut() uses uniswap routers' swap function. Both of these functions do not pass any deadline. `deadline` is used just to make sure the transaction is completed in the required time and does not remain in the mempools for long enough due to low gas fees. Let's say you want to swap TokenA with TokenB. Now as we know transactions sometimes take time to complete, and in such cases, the prices of your tokens might fluctuate a lot, resulting in your inputs not being valid anymore. Thats why its mporant to pass some deadline (user specified or block.timestamp)

### Mitigation
Consider passing a deadline in both Uniswapper.swapExactIn() & Uniswapper.swapExactOut() functions:
```diff
        IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
            .ExactInputParams({
                path: swapParams.path,
                recipient: address(this),
+               //deadline: block.timestamp,
                amountIn: swapParams.amountIn,
                amountOutMinimum: swapParams.amountOut
            });

        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
        amountOut = IV3SwapRouter(uniswap_router).exactInput(params);
```
```diff
        IV3SwapRouter.ExactOutputParams memory params = IV3SwapRouter
            .ExactOutputParams({
                path: swapParams.path,
                recipient: address(this),
-               //deadline: block.timestamp,
+               deadline: block.timestamp,
                amountOut: swapParams.amountOut,
                amountInMaximum: swapParams.amountIn
            });

        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
        amountIn = IV3SwapRouter(uniswap_router).exactOutput(params);
```

## L-05: sgReceive implementation doesn't handle failed situations
If sgReceive fails, it should emitCachedSwapSaved(_srcChainId, _srcAddress, _nonce, reason) and can be retried by calling clearCachedSwap function.

### Mitigation
The sgReceive function should try/catch for failures & emit the CachedSwapSaved. And there should be a `clearCachedSwap()` function in the StartgateRouter to clear the failed messages: https://stargateprotocol.gitbook.io/stargate/stargate-composability/stargatecomposer.sol#sgreceive
