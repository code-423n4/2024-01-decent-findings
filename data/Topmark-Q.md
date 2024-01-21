### Report 1:
#### Wrong Function Argument Handling
The Code provide below shows how some of the arguments in getBridgedAmount(...) function are commented out, the problem is the type is still left in the function even though the protocol don't need it in the function, in addition to this this function was still carried with three parameters as seen at L188 of the UTB.sol contract this is totally wrong and unworthy of the Decent Protocol.
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L75-L76
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L188
```solidity
    function getBridgedAmount(
        uint256 amt2Bridge,
        address /*tokenIn*/,
        address /*tokenOut*/
    ) external pure returns (uint256) {
        return amt2Bridge;
}
```
```solidity
 function swapAndModifyPostBridge(
        BridgeInstructions memory instructions
    )
        private
        returns (
            ...
        newPostSwapParams.amountIn = IBridgeAdapter(
            bridgeAdapters[instructions.bridgeId]
        ).getBridgedAmount(amountOut, tokenOut, newPostSwapParams.tokenIn);
           ...
```
