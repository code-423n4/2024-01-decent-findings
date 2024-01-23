## [L01] Decent is UNLICENSED it's better to choose a proper LICENSE.
`// SPDX-License-Identifier: UNLICENSED`
> all contest


## [L-02] Stop Using Solidity's transfer() Now
**Proof Of Concept**
https://consensys.io/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

```
     IERC20(token).transfer(refund, remainingBalance);
    }
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L80C4-L81C6

```
     function redeemFees(address token, uint amount) public onlyOwner {
        if (token == address(0)) {
            payable(owner).transfer(amount);
        } else {
            IERC20(token).transfer(owner, amount);
        }
    }
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L69C3-L75C6

```
       IERC20(token).transfer(user, amount);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L44C2-L44C46

```
    IERC20(token).transfer(recipient, amount);
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L55C5-L55C51

```
  if (weth.balanceOf(address(this)) < _amount) {
            dcntEth.transfer(_to, _amount);
            return;
        }

        if (msgType == MT_ETH_TRANSFER) {
            if (!gasCurrencyIsEth || !deliverEth) {
                weth.transfer(_to, _amount);
            } else {
                weth.withdraw(_amount);
                payable(_to).transfer(_amount);
            }
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L266C7-L277C14

```
  payable(msg.sender).transfer(amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L290C7-L290C46

```
     weth.transfer(msg.sender, amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L298C4-L298C43

```
     payable(msg.sender).transfer(amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L318C4-L318C46

```
        weth.transfer(msg.sender, amount);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L334C1-L334C43

```
    if (!success) {
            weth.transfer(from, amount);
            return;
        }
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L35C5-L38C10

```
       weth.transfer(from, remainingAfterCall);
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L44C2-L44C49

```
    if (!success) {
            payable(from).transfer(amount);
        }
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L62C5-L64C10

## [L-03] Divide before multiply
Solidity's integer division truncates. Thus, performing division before multiplication can lead to precision loss.

```
 ) external pure returns (uint256) {
        return (amt2Bridge * (1e4 - SG_FEE_BPS)) / 1e4;
    }
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L65C4-L67C6

```
   (amt2Bridge * (10000 - SG_FEE_BPS)) / 10000, // the min qty you would accept on the destination, fee is 6 bips
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L176C10-L176C123


## [L-04]  Empty receive()/payable fallback() function does not authenticate requests
If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))). Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.

```
  receive() external payable {}

    fallback() external payable {}
}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L339C3-L342C2

```
    receive() external payable {}

    fallback() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L77C1-L79C35

```
   receive() external payable {}

    fallback() external payable {}
```
 https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L156C2-L158C35
 
 ```
  receive() external payable {}

    fallback() external payable {}
 ```
 https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L218C4-L220C35
 
```
 receive() external payable {}

    fallback() external payable {}
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L171C4-L173C35

```
 receive() external payable {}

    fallback() external payable {}
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L337C4-L339C35

```
  // Function to receive Ether. msg.data must be empty
    receive() external payable {}

    // Fallback function is called when msg.data is not empty
    fallback() external payable {}
```
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L84C3-L88C35