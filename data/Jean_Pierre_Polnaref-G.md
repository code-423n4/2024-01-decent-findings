# c4udit Report

## Files analyzed
- 2024-01-decent/src/CommonTypes.sol
- 2024-01-decent/src/UTB.sol
- 2024-01-decent/src/UTBExecutor.sol
- 2024-01-decent/src/UTBFeeCollector.sol
- 2024-01-decent/src/UTBOwned.sol
- 2024-01-decent/src/bridge_adapters/BaseAdapter.sol
- 2024-01-decent/src/bridge_adapters/DecentBridgeAdapter.sol
- 2024-01-decent/src/bridge_adapters/StargateBridgeAdapter.sol
- 2024-01-decent/src/bridge_adapters/stargate/IStargateReceiver.sol
- 2024-01-decent/src/bridge_adapters/stargate/IStargateRouter.sol
- 2024-01-decent/src/interfaces/IBridgeAdapter.sol
- 2024-01-decent/src/interfaces/ISwapper.sol
- 2024-01-decent/src/interfaces/IUTB.sol
- 2024-01-decent/src/interfaces/IUTBExecutor.sol
- 2024-01-decent/src/interfaces/IUTBFeeCollector.sol
- 2024-01-decent/src/swappers/SwapParams.sol
- 2024-01-decent/src/swappers/UniSwapper.sol

## Issues found

### Don't Initialize Variables with Default Value

#### Impact
Issue Information: [G001](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g001---dont-initialize-variables-with-default-value)

#### Findings:
```
2024-01-decent/src/UTB.sol::234 => uint value = 0;
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Cache Array Length Outside of Loop

#### Impact
Issue Information: [G002](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g002---cache-array-length-outside-of-loop)

#### Findings:
```
2024-01-decent/src/UTBFeeCollector.sol::29 => require(signature.length == 65, "Invalid signature length");
2024-01-decent/src/swappers/UniSwapper.sol::68 => if (swapParams.path.length == 0) {
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use != 0 instead of > 0 for Unsigned Integer Comparison

#### Impact
Issue Information: [G003](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g003---use--0-instead-of--0-for-unsigned-integer-comparison)

#### Findings:
```
2024-01-decent/src/UTBExecutor.sol::64 => if (extraNative > 0) {
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use immutable for OpenZeppelin AccessControl's Roles Declarations

#### Impact
Issue Information: [G006](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g006---use-immutable-for-openzeppelin-accesscontrols-roles-declarations)

#### Findings:
```
2024-01-decent/src/UTBFeeCollector.sol::49 => bytes32 constructedHash = keccak256(
2024-01-decent/src/UTBFeeCollector.sol::50 => abi.encodePacked(BANNER, keccak256(packedInfo))
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Long Revert Strings

#### Impact
Issue Information: [G007](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g007---long-revert-strings)

#### Findings:
```
2024-01-decent/src/UTB.sol::8 => import {IWETH} from "decent-bridge/src/interfaces/IWETH.sol";
2024-01-decent/src/UTB.sol::9 => import {IUTBFeeCollector} from "./interfaces/IUTBFeeCollector.sol";
2024-01-decent/src/bridge_adapters/BaseAdapter.sol::14 => "Only bridge executor can call this"
2024-01-decent/src/bridge_adapters/DecentBridgeAdapter.sol::9 => import {IDecentEthRouter} from "decent-bridge/src/interfaces/IDecentEthRouter.sol";
2024-01-decent/src/swappers/UniSwapper.sol::8 => import {IWETH} from "decent-bridge/src/interfaces/IWETH.sol";
2024-01-decent/src/swappers/UniSwapper.sol::11 => import {IV3SwapRouter} from "@uniswap/swap-contracts/interfaces/IV3SwapRouter.sol";
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use Shift Right/Left instead of Division/Multiplication if possible

#### Impact
Issue Information: [G008](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md/#g008---use-shift-rightleft-instead-of-divisionmultiplication-if-possible)

#### Findings:
```
2024-01-decent/src/bridge_adapters/DecentBridgeAdapter.sol::75 => address /*tokenIn*/,
2024-01-decent/src/bridge_adapters/StargateBridgeAdapter.sol::63 => address /*tokenIn*/,
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Unsafe ERC20 Operation(s)

#### Impact
Issue Information: [L001](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l001---unsafe-erc20-operations)

#### Findings:
```
2024-01-decent/src/UTB.sol::83 => IERC20(swapParams.tokenIn).transferFrom(
2024-01-decent/src/UTB.sol::90 => IERC20(swapParams.tokenIn).approve(
2024-01-decent/src/UTB.sol::152 => IERC20(tokenOut).approve(address(executor), amountOut);
2024-01-decent/src/UTB.sol::216 => IERC20(bridgeToken).approve(address(bridgeAdapter), amt2Bridge);
2024-01-decent/src/UTB.sol::236 => IERC20(fees.feeToken).transferFrom(
2024-01-decent/src/UTB.sol::241 => IERC20(fees.feeToken).approve(
2024-01-decent/src/UTBExecutor.sol::61 => IERC20(token).transferFrom(msg.sender, address(this), amount);
2024-01-decent/src/UTBExecutor.sol::62 => IERC20(token).approve(paymentOperator, amount);
2024-01-decent/src/UTBExecutor.sol::80 => IERC20(token).transfer(refund, remainingBalance);
2024-01-decent/src/UTBFeeCollector.sol::56 => IERC20(fees.feeToken).transferFrom(
2024-01-decent/src/UTBFeeCollector.sol::71 => payable(owner).transfer(amount);
2024-01-decent/src/UTBFeeCollector.sol::73 => IERC20(token).transfer(owner, amount);
2024-01-decent/src/bridge_adapters/DecentBridgeAdapter.sol::109 => IERC20(bridgeToken).transferFrom(
2024-01-decent/src/bridge_adapters/DecentBridgeAdapter.sol::114 => IERC20(bridgeToken).approve(address(router), amt2Bridge);
2024-01-decent/src/bridge_adapters/DecentBridgeAdapter.sol::139 => IERC20(swapParams.tokenIn).transferFrom(
2024-01-decent/src/bridge_adapters/DecentBridgeAdapter.sol::145 => IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn);
2024-01-decent/src/bridge_adapters/StargateBridgeAdapter.sol::88 => IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);
2024-01-decent/src/bridge_adapters/StargateBridgeAdapter.sol::89 => IERC20(bridgeToken).approve(address(router), amt2Bridge);
2024-01-decent/src/bridge_adapters/StargateBridgeAdapter.sol::207 => IERC20(swapParams.tokenIn).approve(utb, swapParams.amountIn);
2024-01-decent/src/swappers/UniSwapper.sol::44 => IERC20(token).transfer(user, amount);
2024-01-decent/src/swappers/UniSwapper.sol::55 => IERC20(token).transfer(recipient, amount);
2024-01-decent/src/swappers/UniSwapper.sol::83 => IERC20(swapParams.tokenIn).transferFrom(
2024-01-decent/src/swappers/UniSwapper.sol::137 => IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
2024-01-decent/src/swappers/UniSwapper.sol::158 => IERC20(swapParams.tokenIn).approve(uniswap_router, swapParams.amountIn);
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Unspecific Compiler Version Pragma

#### Impact
Issue Information: [L003](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l003---unspecific-compiler-version-pragma)

#### Findings:
```
2024-01-decent/src/CommonTypes.sol::2 => pragma solidity ^0.8.0;
2024-01-decent/src/UTB.sol::2 => pragma solidity ^0.8.0;
2024-01-decent/src/UTBExecutor.sol::2 => pragma solidity ^0.8.0;
2024-01-decent/src/UTBFeeCollector.sol::2 => pragma solidity ^0.8.0;
2024-01-decent/src/UTBOwned.sol::2 => pragma solidity ^0.8.0;
2024-01-decent/src/bridge_adapters/BaseAdapter.sol::2 => pragma solidity ^0.8.0;
2024-01-decent/src/bridge_adapters/DecentBridgeAdapter.sol::2 => pragma solidity ^0.8.0;
2024-01-decent/src/bridge_adapters/StargateBridgeAdapter.sol::2 => pragma solidity ^0.8.0;
2024-01-decent/src/bridge_adapters/stargate/IStargateReceiver.sol::2 => pragma solidity ^0.8.0;
2024-01-decent/src/bridge_adapters/stargate/IStargateRouter.sol::2 => pragma solidity ^0.8.0;
2024-01-decent/src/interfaces/IBridgeAdapter.sol::1 => pragma solidity ^0.8.0;
2024-01-decent/src/interfaces/ISwapper.sol::1 => pragma solidity ^0.8.0;
2024-01-decent/src/interfaces/IUTB.sol::1 => pragma solidity ^0.8.0;
2024-01-decent/src/interfaces/IUTBExecutor.sol::1 => pragma solidity ^0.8.0;
2024-01-decent/src/interfaces/IUTBFeeCollector.sol::1 => pragma solidity ^0.8.0;
2024-01-decent/src/swappers/SwapParams.sol::2 => pragma solidity ^0.8.0;
2024-01-decent/src/swappers/UniSwapper.sol::2 => pragma solidity ^0.8.0;
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)
