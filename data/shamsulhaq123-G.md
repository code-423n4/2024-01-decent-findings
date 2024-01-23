## Gas Optimization

## Summary

|No|Issue|Instance|
|--|-----|--------|
|[G-01]Use hardcode address instead address(this)|23|
|[G-02]+= costs more gas than = + for state variables|1|
|[G-03]Change public function visibility to external|35|
|[G-04]Unnecessary look up in if condition|2|
|[G-05]Use != 0 instead of > 0 for unsigned integer comparison|1|
|[G-06]abi.encode() is less efficient than abi.encodePacked()|6|
|[G-07]Empty Blocks Should Be Removed Or Emit Something|7|
|[G-08] Use assembly in place of abi.decode to extract calldata values more efficiently|6|

## [G-01] Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address.

```solidity

file: UTB.sol

 IERC20(swapParams.tokenIn).transferFrom(
85                msg.sender,
                address(this),
                swapParams.amountIn
            );
238     msg.sender,
                    address(this),
                    fees.feeAmount
                );            
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L85
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L238

```solidity

file: UTBExecutor.sol

59  uint initBalance = IERC20(token).balanceOf(address(this));

61        IERC20(token).transferFrom(msg.sender, address(this), amount);

73   uint remainingBalance = IERC20(token).balanceOf(address(this)) -
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L59-L61
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L73

```solidity

file: UTBFeeCollector.sol

56  IERC20(fees.feeToken).transferFrom(
                utb,
                address(this),
                fees.feeAmount

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L56

```solidity

file: bridge_adapters/DecentBridgeAdapter.so

111  IERC20(bridgeToken).transferFrom(
                msg.sender,
                address(this),
                amt2Bridge
            );

141 IERC20(swapParams.tokenIn).transferFrom(
            msg.sender,
            address(this),
            swapParams.amountIn
        );
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L111
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L141

```solidity

file: bridge_adapters/StargateBridgeAdapter.sol

88   IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88

```solidity

file: swappers/UniSwapper.sol

83  IERC20(swapParams.tokenIn).transferFrom(
                msg.sender,
                address(this),

132 .ExactInputParams({
                path: swapParams.path,
                recipient: address(this),

50    .ExactOutputParams({
                path: swapParams.path,
                recipient: address(this),                
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L83
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L132
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L150

```solidity

file: DecentEthRouter.sol

51  require(weth.balanceOf(address(this)) > amount, "not enough reserves");

181  weth.transferFrom(msg.sender, address(this), _amount);

266  if (weth.balanceOf(address(this)) < _amount) 

288  dcntEth.transferFrom(msg.sender, address(this), amount);

297   dcntEth.transferFrom(msg.sender, address(this), amount);

309  dcntEth.mint(address(this), msg.value);

316  dcntEth.burn(address(this), amount);

325   weth.transferFrom(msg.sender, address(this), amount);
       dcntEth.mint(address(this), amount);
333   dcntEth.burn(address(this), amount);       
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L51
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L181
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L266
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L288
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L297
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L309
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L316
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L325
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L333

```solidity

file: DecentBridgeExecutor.sol

30    uint256 balanceBefore = weth.balanceOf(address(this));

41   (balanceBefore - weth.balanceOf(address(this)));

75   weth.transferFrom(msg.sender, address(this), amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L30
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L41
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L75

## [G-02] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: DecentEthRouter.sol

56      balanceOf[msg.sender] += amount;
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L56

## [G-03] Change public function visibility to external
Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

```solidity

file:UTB.sol

28  function setExecutor(address _executor) public onlyOwner 

36     function setWrapped(address payable _wrapped) public onlyOwner

44 function setFeeCollector(address payable _feeCollector) public onlyOwner 

108 function swapAndExecute(
        SwapAndExecuteInstructions calldata instructions,
        FeeStructure calldata fees,
        bytes calldata signature
    )
        public
259  function bridgeAndExecute(
        BridgeInstructions calldata instructions,
        FeeStructure calldata fees,
        bytes calldata signature
    )
        public      

311  function receiveFromBridge(
        SwapInstructions memory postBridge,
        address target,
        address paymentOperator,
        bytes memory payload,
        address payable refund
    ) public     

325   function registerSwapper(address swapper) public onlyOwner     

334  function registerBridge(address bridge) public onlyOwner x
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L28
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L36
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L44
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L108
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L259
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L311
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L325
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L334

```solidity

file: UTBExecutor.sol

19  function execute(
        address target,
        address paymentOperator,
        bytes memory payload,
        address token,
        uint amount,
        address payable refund
    ) public payable onlyOwner

41  function execute(
        address target,
        address paymentOperator,
        bytes memory payload,
        address token,
        uint amount,
        address payable refund,
        uint extraNative
    ) public onlyOwner
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L19
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L41

```solidity

file: UTBFeeCollector.sol

18  function setSigner(address _signer) public onlyOwner 

48 function collectFees(
        FeeStructure calldata fees,
        bytes memory packedInfo,
        bytes memory signature
    ) public payable onlyUtb 

69  function redeemFees(address token, uint amount) public onlyOwner     
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L18
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L48
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L69

```solidity

file: bridge_adapters/BaseAdapter.sol

19  function setBridgeExecutor(address _executor) public onlyOwner
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L19

```solidity

file: bridge_adapters/DecentBridgeAdapter.sol

26   function setRouter(address _router) public onlyOwner

30    function getId() public pure returns (uint8) 

34 
    function registerRemoteBridgeAdapter(
        uint256 dstChainId,
        uint16 dstLzId,
        address decentBridgeAdapter
    ) public onlyOwner 

44 
    function estimateFees(
        SwapInstructions memory postBridge,
        uint256 dstChainId,
        address target,
        uint64 dstGas,
        bytes memory payload
    ) public view returns (uint nativeFee, uint zroFee)   

81  function bridge(
        uint256 amt2Bridge,
        SwapInstructions memory postBridge,
        uint256 dstChainId,
        address target,
        address paymentOperator,
        bytes memory payload,
        bytes calldata additionalArgs,
        address payable refund
    ) public payable onlyUtb returns (bytes memory bridgePayload) 

127   function receiveFromBridge(
        SwapInstructions memory postBridge,
        address target,
        address paymentOperator,
        bytes memory payload,
        address payable refund
    ) public onlyExecutor        
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L26
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L30
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L34
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L44
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L81
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L127

```solidity

file: bridge_adapters/StargateBridgeAdapter.sol

33  function setRouter(address _router) public onlyOwner

37  function setStargateEth(address _sgEth) public onlyOwner {
        stargateEth = _sgEth;
    }

41   function getId() public pure returns (uint8) {
        return BRIDGE_ID;
    }    
45 
    function registerRemoteBridgeAdapter(
        uint256 dstChainId,
        uint16 dstLzId,
        address decentBridgeAdapter
    ) public onlyOwner   
69 
    function bridge(
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
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L33
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L37
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L41
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L45
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L69

```solidity

file: swappers/UniSwapper.sol

20   function setRouter(address _router) public onlyOwner {
        uniswap_router = _router;
    }

24   function setWrapped(address payable _wrapped) public onlyOwner {
        wrapped = _wrapped;
    }
28 
    function getId() public pure returns (uint8) {
        return SWAPPER_ID;
    }    
100  function swapNoPath(
        SwapParams memory swapParams,
        address receiver,
        address refund
    ) public payable returns (address tokenOut, uint256 amountOut)   

123   function swapExactIn(
        SwapParams memory swapParams, // SwapParams is a struct
        address receiver
    ) public payable routerIsSet returns (uint256 amountOut)  
143 
    function swapExactOut(
        SwapParams memory swapParams,
        address receiver,
        address refundAddress
    ) public payable routerIsSet returns (uint256 amountIn)          
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L20
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L24
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L28
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L100
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L123
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L143

```solidity

file: DcntEth.sol
 
20   function setRouter(address _router) public {
        router = _router;
    }

    function mint(address _to, uint256 _amount) public onlyRouter {
        _mint(_to, _amount);
    }

    function burn(address _from, uint256 _amount) public onlyRouter {
        _burn(_from, _amount);
    }

    function mintByOwner(address _to, uint256 _amount) public onlyOwner {
        _mint(_to, _amount);
    }

36   function burnByOwner(address _from, uint256 _amount) public onlyOwner {
        _burn(_from, _amount);
   }
}
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L20-L36

```solidity

file: DecentEthRouter.sol

68   function registerDcntEth(address _addr) public onlyOwner {
        dcntEth = IDcntEth(_addr);
    }
73    function addDestinationBridge(
        uint16 _dstChainId,
        address _routerAddress
    ) public onlyOwner   

113  function estimateSendAndCallFee(
        uint8 msgType,
        uint16 _dstChainId,
        address _toAddress,
        uint _amount,
        uint64 _dstGasForCall,
        bool deliverEth,
        bytes memory payload
    ) public view returns (uint nativeFee, uint zroFee)     
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L68
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L73
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L113

# [G-04] Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: DecentEthRouter.sol

272  if (!gasCurrencyIsEth || !deliverEth)
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L272

```solidity

file: DecentBridgeExecutor.sol

77  if (!gasCurrencyIsEth || !deliverEth)

```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L77

## [G-05] Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

```solidity

file: UTBExecutor.sol

64 if (extraNative > 0) 
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L64

## [G-06] abi.encode() is less efficient than abi.encodePacked()

```solidity

file: UTB.sol

115   retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)

266       retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L115
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L266

```solidity

file: bridge_adapters/StargateBridgeAdapter.sol

69     bridgePayload = abi.encode(

179  bridgePayload // bytes param, if you wish to send additional payload you can abi.encode() 
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L69
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L179

```solidity

file: swappers/UniSwapper.sol

40   return abi.encode(newSwapParams, receiver, refund);

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L40

```solidity

file: DecentEthRouter.sol

99  destBridge = bytes32(abi.encode(destinationBridges[_dstChainId]));
        if (msgType == MT_ETH_TRANSFER) {
            payload = abi.encode(msgType, msg.sender, _toAddress, deliverEth);
        } else {
            payload = abi.encode(
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L99-L103

## [G-07] Empty Blocks Should Be Removed Or Emit Something
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){…}else{…} => if(!x){if(y){…}else{…}})

```solidity

file: UTB.sol

339  receive() external payable {}

341    fallback() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L339-341

```solidity

file: UTBFeeCollector.sol

77 
    receive() external payable {}

    fallback() external payable {}

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L77-L79

```solidity

file: bridge_adapters/DecentBridgeAdapter.sol

156    receive() external payable {}

    fallback() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L156-158

```solidity

file: bridge_adapters/StargateBridgeAdapter.sol

218  receive() external payable {}

220  fallback() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L218-L220

```solidity

file: swappers/UniSwapper.sol

171    receive() external payable {}

173    fallback() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L171-L173

```solidity

file: DecentEthRouter.sol

337     receive() external payable {}

339    fallback() external payable {}

```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L337-L339

```solidity

file: DecentBridgeExecutor.sol

85   receive() external payable {}

88        fallback() external payable {}
}
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L85-L88

## [G-08] Use assembly in place of abi.decode to extract calldata values more efficiently
Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

```solidity

file: UTB.sol

115   retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)

266       retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L115
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L266

```solidity

file: bridge_adapters/StargateBridgeAdapter.sol

69     bridgePayload = abi.encode(

179  bridgePayload // bytes param, if you wish to send additional payload you can abi.encode() 
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L69
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L179

```solidity

file: swappers/UniSwapper.sol

40   return abi.encode(newSwapParams, receiver, refund);

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L40

```solidity

file: DecentEthRouter.sol

99  destBridge = bytes32(abi.encode(destinationBridges[_dstChainId]));
        if (msgType == MT_ETH_TRANSFER) {
            payload = abi.encode(msgType, msg.sender, _toAddress, deliverEth);
        } else {
            payload = abi.encode(
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L99-L103



```
