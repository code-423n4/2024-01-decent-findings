# Gas Optimization

# Summary

| Number | Gas Optimization                                                                      | Context |
| :----: | :------------------------------------------------------------------------------------ | :-----: |
| [G-01] | Using fixed BYTES CONSTANTS is cheaper than using string                              |    1    |
| [G-02] | Avoid contract existence checks by using low level calls                              |   68    |
| [G-03] | Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4 |    3    |
| [G-04] | Use ASSEMBLY to hash instead of solidity                                              |    2    |
| [G-05] | abi.encode() is less efficient than abi.encodePacked()                                |    7    |
| [G-06] | Using assembly to revert with an error message                                        |   12    |
| [G-07] | Using constants directly, rather than caching the value, saves gas                    |    3    |
| [G-08] | Using >0 costs more gas than != 0                                                     |    1    |
| [G-09] | Use ASSEMBLY to perform efficient back-to-back calls                                  |   19    |
| [G-10] | Use hardcode address instead address(this)                                            |   14    |
| [G-11] | <X> += <Y> costs more gas than  <X> = <X> + <Y> for state variables                   |    1    |

## [G-01] Using fixed BYTES CONSTANTS is cheaper than using string

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

```solidity
File: main/src/UTBFeeCollector.sol

10    string constant BANNER = "\x19Ethereum Signed Message:\n32";
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L10

## [G-02] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

56            router.estimateSendAndCallFee(

57                router.MT_ETH_TRANSFER_WITH_PAYLOAD(),

109            IERC20(bridgeToken).transferFrom(

114            IERC20(bridgeToken).approve(address(router), amt2Bridge);

139        IERC20(swapParams.tokenIn).transferFrom(

145        IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn);

147        IUTB(utb).receiveFromBridge(
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L56

```solidity
File: src/DecentBridgeExecutor.sol

30        uint256 balanceBefore = weth.balanceOf(address(this));

31        weth.approve(target, amount);

36            weth.transfer(from, amount);

41            (balanceBefore - weth.balanceOf(address(this)));

44        weth.transfer(from, remainingAfterCall);

60        weth.withdraw(amount);

63            payable(from).transfer(amount);
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L30

```solidity
File: src/DecentEthRouter.sol

178            weth.deposit{value: _amount}();

181            weth.transferFrom(msg.sender, address(this), _amount);

185        dcntEth.sendAndCall{value: gasValue}(

266        if (weth.balanceOf(address(this)) < _amount) {

267            dcntEth.transfer(_to, _amount);

273                weth.transfer(_to, _amount);

275                weth.withdraw(_amount);

276                payable(_to).transfer(_amount);

279            weth.approve(address(executor), _amount);

280            executor.execute(_from, _to, deliverEth, _amount, callPayload);

288        dcntEth.transferFrom(msg.sender, address(this), amount);

289        weth.withdraw(amount);

290        payable(msg.sender).transfer(amount);

297        dcntEth.transferFrom(msg.sender, address(this), amount);

298        weth.transfer(msg.sender, amount);

308        weth.deposit{value: msg.value}();

309        dcntEth.mint(address(this), msg.value);

316        dcntEth.burn(address(this), amount);

317        weth.withdraw(amount);

318        payable(msg.sender).transfer(amount);

325        weth.transferFrom(msg.sender, address(this), amount);

326        dcntEth.mint(address(this), amount);

333        dcntEth.burn(address(this), amount);

334        weth.transfer(msg.sender, amount);
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L178

```solidity
File: src/swappers/UniSwapper.sol

137        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);

138        amountOut = IV3SwapRouter(uniswap_router).exactInput(params);

158        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);

159        amountIn = IV3SwapRouter(uniswap_router).exactOutput(params);
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L137

```solidity
File: src/UTB.sol

76            wrapped.deposit{value: swapParams.amountIn}();

78            swapInstructions.swapPayload = swapper.updateSwapParams(

83            IERC20(swapParams.tokenIn).transferFrom(

90        IERC20(swapParams.tokenIn).approve(

95        (tokenOut, amountOut) = swapper.swap(swapInstructions.swapPayload);

98            wrapped.withdraw(amountOut);

143            executor.execute{value: amountOut}(

152            IERC20(tokenOut).approve(address(executor), amountOut);

153            executor.execute(

236                IERC20(fees.feeToken).transferFrom(

241                IERC20(fees.feeToken).approve(

248            feeCollector.collectFees{value: value}(fees, packedInfo, signature);
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L76

```solidity
File:  src/UTBExecutor.sol

59        uint initBalance = IERC20(token).balanceOf(address(this));

61        IERC20(token).transferFrom(msg.sender, address(this), amount);

62        IERC20(token).approve(paymentOperator, amount);

73        uint remainingBalance = IERC20(token).balanceOf(address(this)) -
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L52

## [G-03] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

```solidity
File: src/DecentEthRouter.sol

98        adapterParams = abi.encodePacked(PT_SEND_AND_CALL, gasAmount);
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L98

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

178            abi.encodePacked(getDestAdapter(dstChainId)),

```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L178

```solidity
File: src/UTBFeeCollector.sol

50            abi.encodePacked(BANNER, keccak256(packedInfo))

```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L50

## [G-04] Use ASSEMBLY to hash instead of solidity

Use ASSEMBLY for small keccak256 hashes, in order to save gas

If the arguments to the encode call can fit into the scratch space (two words or fewer),

then it's more efficient to use assembly to generate the hash (80 gas):

keccak256(abi.encodePacked(x, y)) -> assembly {mstore(0x00, a); mstore(0x20, b); let hash := keccak256(0x00, 0x40); }

There is one instance of this issue: keccak256(abi.encode(\_operation))

    newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(_user, address(this));

```solidity
File: src/UTBFeeCollector.sol

49        bytes32 constructedHash = keccak256(
50            abi.encodePacked(BANNER, keccak256(packedInfo))
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L49

## [G-05] abi.encode() is less efficient than abi.encodePacked()

These are missed form bot

```solidity
File: src/DecentEthRouter.sol

99        destBridge = bytes32(abi.encode(destinationBridges[_dstChainId]));

101            payload = abi.encode(msgType, msg.sender, _toAddress, deliverEth);

103            payload = abi.encode(
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L99

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

81        bridgePayload = abi.encode(
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L81

```solidity
File: src/swappers/UniSwapper.sol

40        return abi.encode(newSwapParams, receiver, refund);
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L40

```solidity
File: src/UTB.sol

115        retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)

266        payable
        retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L115

## [G-06] Using assembly to revert with an error message

When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

```solidity
File: src/bridge_adapters/BaseAdapter.sol

12        require(
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L12

```solidity
File: src/DcntEth.sol

9        require(msg.sender == router);
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L9

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

91        require(
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L19

```solidity
File: src/DecentEthRouter.sol

38        require(gasCurrencyIsEth, "Gas currency is not ETH");

43        require(

51        require(weth.balanceOf(address(this)) > amount, "not enough reserves");

62        require(balance >= amount, "not enough balance");
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L38

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

157        require(
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L81

```solidity
File: src/swappers/UniSwapper.sol

96        require(uniswap_router != address(0), "router not set");
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L137

```solidity
File: src/UTB.sol

75            require(msg.value >= swapParams.amountIn, "not enough native");
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L76

```solidity
File: src/UTBFeeCollector.sol

29        require(signature.length == 65, "Invalid signature length");

54        require(recovered == signer, "Wrong signature");
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L10

## [G-07] Using constants directly, rather than caching the value, saves gas

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

30    function getId() public pure returns (uint8) {
31        return BRIDGE_ID;
32    }

```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L30

```solidity
File: src/bridge_adapters/StargateBridgeAdapter.sol

41    function getId() public pure returns (uint8) {
42        return BRIDGE_ID;
43    }
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L41

```solidity
File: src/swappers/UniSwapper.sol

28    function getId() public pure returns (uint8) {
29        return SWAPPER_ID;
30    }
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L28-L30

## [G-08] Using >0 costs more gas than != 0

```solidity
File: src/UTBExecutor.sol

64        if (extraNative > 0) {
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L64

## [G-09] Use ASSEMBLY to perform efficient back-to-back calls

Optimize External Calls with ASSEMBLY for Memory Efficiency

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

56            router.estimateSendAndCallFee(
57                router.MT_ETH_TRANSFER_WITH_PAYLOAD(),


109            IERC20(bridgeToken).transferFrom(
114            IERC20(bridgeToken).approve(address(router), amt2Bridge);
117        router.bridgeWithPayload{value: msg.value}(


139        IERC20(swapParams.tokenIn).transferFrom(
145        IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn);
147        IUTB(utb).receiveFromBridge(
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L56

```solidity
File: src/DecentBridgeExecutor.sol

30        uint256 balanceBefore = weth.balanceOf(address(this));
31        weth.approve(target, amount);
36            weth.transfer(from, amount);
41            (balanceBefore - weth.balanceOf(address(this)));
44        weth.transfer(from, remainingAfterCall);

60        weth.withdraw(amount);
61        (bool success, ) = target.call{value: amount}(callPayload);
63            payable(from).transfer(amount);
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L30

```solidity
File: src/DecentEthRouter.sol

178            weth.deposit{value: _amount}();
181            weth.transferFrom(msg.sender, address(this), _amount);
185        dcntEth.sendAndCall{value: gasValue}(


266        if (weth.balanceOf(address(this)) < _amount) {
267            dcntEth.transfer(_to, _amount);
273                weth.transfer(_to, _amount);
275                weth.withdraw(_amount);
276                payable(_to).transfer(_amount);
279            weth.approve(address(executor), _amount);
280            executor.execute(_from, _to, deliverEth, _amount, callPayload);


288        dcntEth.transferFrom(msg.sender, address(this), amount);
289        weth.withdraw(amount);
290        payable(msg.sender).transfer(amount);


297        dcntEth.transferFrom(msg.sender, address(this), amount);
298        weth.transfer(msg.sender, amount);

308        weth.deposit{value: msg.value}();
309        dcntEth.mint(address(this), msg.value);


316        dcntEth.burn(address(this), amount);
317        weth.withdraw(amount);
318        payable(msg.sender).transfer(amount);


325        weth.transferFrom(msg.sender, address(this), amount);
326        dcntEth.mint(address(this), amount);


333        dcntEth.burn(address(this), amount);
334        weth.transfer(msg.sender, amount);
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L178

```solidity
File: src/swappers/UniSwapper.sol

137        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
138        amountOut = IV3SwapRouter(uniswap_router).exactInput(params);

158        IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
159        amountIn = IV3SwapRouter(uniswap_router).exactOutput(params);
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L137

```solidity
File: src/UTB.sol

76            wrapped.deposit{value: swapParams.amountIn}();
78            swapInstructions.swapPayload = swapper.updateSwapParams(
83            IERC20(swapParams.tokenIn).transferFrom(
90        IERC20(swapParams.tokenIn).approve(
95        (tokenOut, amountOut) = swapper.swap(swapInstructions.swapPayload);
98            wrapped.withdraw(amountOut);


143            executor.execute{value: amountOut}(
152            IERC20(tokenOut).approve(address(executor), amountOut);
153            executor.execute(


236                IERC20(fees.feeToken).transferFrom(
241                IERC20(fees.feeToken).approve(
248            feeCollector.collectFees{value: value}(fees, packedInfo, signature);
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L76

```solidity
File:  src/UTBExecutor.sol

52            (success, ) = target.call{value: amount}(payload);
54                (refund.call{value: amount}(""));
59        uint initBalance = IERC20(token).balanceOf(address(this));
61        IERC20(token).transferFrom(msg.sender, address(this), amount);
62        IERC20(token).approve(paymentOperator, amount);
65            (success, ) = target.call{value: extraNative}(payload);
67                (refund.call{value: extraNative}(""));
73        uint remainingBalance = IERC20(token).balanceOf(address(this)) -
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L52

## [G-10] Use hardcode address instead address(this)

```solidity
File: src/bridge_adapters/DecentBridgeAdapter.sol

111                address(this),

141            address(this),
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L111

```solidity
File: src/DecentBridgeExecutor.sol

30        uint256 balanceBefore = weth.balanceOf(address(this));

41            (balanceBefore - weth.balanceOf(address(this)));

75        weth.transferFrom(msg.sender, address(this), amount);
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L30

```solidity
File: src/DecentEthRouter.sol

51        require(weth.balanceOf(address(this)) > amount, "not enough reserves");

181            weth.transferFrom(msg.sender, address(this), _amount);

186            address(this), // from address that has dcntEth (so DecentRouter)

266        if (weth.balanceOf(address(this)) < _amount) {

288        dcntEth.transferFrom(msg.sender, address(this), amount);

297        dcntEth.transferFrom(msg.sender, address(this), amount);

309        dcntEth.mint(address(this), msg.value);

316        dcntEth.burn(address(this), amount);


325        weth.transferFrom(msg.sender, address(this), amount);

333        dcntEth.burn(address(this), amount);

```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L51

```solidity
File: src/swappers/UniSwapper.sol

85                address(this),

132                recipient: address(this),

152                recipient: address(this),
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L85

```solidity
File: src/UTB.sol

85                address(this),

238                    address(this),
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L85

```solidity
File: src/UTBExecutor.sol

59        uint initBalance = IERC20(token).balanceOf(address(this));

61        IERC20(token).transferFrom(msg.sender, address(this), amount);

73        uint remainingBalance = IERC20(token).balanceOf(address(this))
```

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L59

## [G-11] <X> += <Y> costs more gas than  <X> = <X> + <Y> for state variables

```solidity
File: src/DecentEthRouter.sol

56        balanceOf[msg.sender] += amount;
```

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L56