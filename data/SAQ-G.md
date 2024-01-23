## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Avoid contract existence checks by using low level calls | 6 | - |
| [G-02] | Multiplication/division by two should use bit shifting | 1 | - |
| [G-03] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= ) | 2 | - |
| [G-04] | Before transfer of  some functions, we should check some variables for possible gas save | 7 | - |
| [G-05] | Bytes constants are more efficient than String constants | 1 | - |
| [G-06] | Empty blocks should be removed or emit something | 6 | - |

## Gas Optimizations  

## [G-01] Avoid contract existence checks by using low level calls		

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.			


```solidity
file: /src/UTB.sol

78            swapInstructions.swapPayload = swapper.updateSwapParams(
                swapParams,
                swapInstructions.swapPayload
            );


83            IERC20(swapParams.tokenIn).transferFrom(
                msg.sender,
                address(this),
                swapParams.amountIn
            );
        }           


95        (tokenOut, amountOut) = swapper.swap(swapInstructions.swapPayload);


218            feeCollector.collectFees{value: value}(fees, packedInfo, signature);


```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L78C1-L81C15


```solidity
file: /UTBExecutor.sol

52            (success, ) = target.call{value: amount}(payload);

73        uint remainingBalance = IERC20(token).balanceOf(address(this)) -
            initBalance;


```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L52C1-L52C63


## [G-02] Multiplication/division by two should use bit shifting

<x> * 2 is equivalent to <x> << 1 

and <x> / 2 is the same as <x> >> 1. 

```solidity
file: /bridge_adapters/StargateBridgeAdapter.sol

66        return (amt2Bridge * (1e4 - SG_FEE_BPS)) / 1e4;

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L66C1-L66C56


## [G-03] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= )


AVOID COMPOUND ASSIGNMENT OPERATOR IN STATE VARIABLES


```solidity
file: /DecentEthRouter.sol

56        balanceOf[msg.sender] += amount;

64        balanceOf[msg.sender] -= amount;

```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L56C1-L56C41


## [G-04] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:

```solidity
file: /swappers/UniSwapper.sol

44        IERC20(token).transfer(user, amount);

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L44C1-L44C46


```solidity
file: /bridge_adapters/DecentBridgeAdapter.sol

139        IERC20(swapParams.tokenIn).transferFrom(
            msg.sender,
            address(this),
            swapParams.amountIn
        );

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L139C2-L143C11


```solidity
file: /bridge_adapters/StargateBridgeAdapter.sol

88        IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88C1-L88C81


```solidity
file: /DecentEthRouter.sol

288        dcntEth.transferFrom(msg.sender, address(this), amount);

290        payable(msg.sender).transfer(amount);

297        dcntEth.transferFrom(msg.sender, address(this), amount);

```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L288C1-L288C65


```solidity
file: /DecentBridgeExecutor.sol

44        weth.transfer(from, remainingAfterCall);

```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L44C1-L44C49


## [G-05] Bytes constants are more efficient than String constants

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.


```solidity
file: /UTBFeeCollector.sol

10    string constant BANNER = "\x19Ethereum Signed Message:\n32";

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L10C1-L10C65


## [G-06] Empty blocks should be removed or emit something

The gas cost of an empty constructor block in Solidity is typically in the range of 200-500 gas units. 

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

```solidity
file: /src/UTB.sol

339    receive() external payable {}

341    fallback() external payable {}

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L339C1-L341C35


```solidity
file: /src/UTBFeeCollector.sol

77    receive() external payable {}

79    fallback() external payable {}

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L77C1-L79C35


```solidity
file: /bridge_adapters/DecentBridgeAdapter.sol

156    receive() external payable {}

158    fallback() external payable {}

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L156C1-L158C35
