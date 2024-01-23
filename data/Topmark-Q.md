### Report 1:
#### Inconsistent Decoding
The the type arrangements used to decode `additionalArgs` variable is inconsistent in the DecentBridgeAdapter & StargateBridgeAdapter contracts. Since additionalArgs is the same variable as can be seen in the functions below, a level of consistency is expected in the variable types that can be decoded but this was not the case in both contract which opens up the risk of wrong return values.
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L96
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L58
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L104
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L117
```solidity
       function bridge(
        ... 
        bytes calldata additionalArgs,
        address payable refund
    ) public payable onlyUtb returns (bytes memory bridgePayload) {
        require(
            destinationBridgeAdapter[dstChainId] != address(0),
            string.concat("dst chain address not set ")
        );

>>>        uint64 dstGas = abi.decode(additionalArgs, (uint64));
```
```solidity
     function getBridgeToken(
        bytes calldata additionalArgs
    ) external pure returns (address bridgeToken) {
>>>        bridgeToken = abi.decode(additionalArgs, (address));
    }
```
```solidity
      function getValue(
        bytes calldata additionalArgs,
        uint256 amt2Bridge
    ) private view returns (uint value) {
>>>        (address bridgeToken, LzBridgeData memory lzBridgeData) = abi.decode(
            additionalArgs,
            (address, LzBridgeData)
        );
       ...
    }
```
```solidity
    function getLzTxObj(
        bytes calldata additionalArgs
    ) private pure returns (IStargateRouter.lzTxObj memory) {
>>>        (, , IStargateRouter.lzTxObj memory lzTxObj) = abi.decode(
            additionalArgs,
            (address, LzBridgeData, IStargateRouter.lzTxObj)
        );
        return lzTxObj;
    } 
```
Note: uint64 was not put into consideration when decoding this, it was put into consideration only in the first function.
### Report 2:
#### Wrong Variable Representation
The _bridgeWithPayload(...) function below in the DecentEthRouter.sol contract seems to misrepresent the actual implementation of the code, The condition is checking if gas Currency is Eth but the implementation inside both conditional statement shows gas is handled as Eth in both cases either the protocol meant to say `if (CurrencyIsEth)` instead of `if (gasCurrencyIsEth)`. If that is not the case which means there is an error in the code implementation and this report would be of higher severity as gas was handled as a part or whole of msg.value in both cases. Protocol should ensure necessary adjustments are made.
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L176-L183
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#22
```solidity
   function _bridgeWithPayload(
        uint8 msgType,
        uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        uint64 _dstGasForCall,
        bytes memory additionalPayload,
        bool deliverEth
    ) internal {
        ...

        uint gasValue;
>>>        if (gasCurrencyIsEth) {
            weth.deposit{value: _amount}();
>>>           gasValue = msg.value - _amount;
        } else {
            weth.transferFrom(msg.sender, address(this), _amount);
>>>            gasValue = msg.value;

>>> dcntEth.sendAndCall{value: gasValue}(
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
```solidity
  bool public gasCurrencyIsEth; // for chains that use ETH as gas currency
```
### Report 3:
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

