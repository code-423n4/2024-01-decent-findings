## [L-01] Remove unused `getValue()` function from `StargateBridgeAdapter.sol` contract
The `StargateBridgeAdapter::getValue()` function is not used anywhere in the contract or codebase and should be removed to prevent conflicts arising from further development of the codebase that may introduce functions or logic conflicting with the `getValue()` unused function

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L100-L122

```solidity
  function getValue( // unused function @audit-info
      bytes calldata additionalArgs,
      uint256 amt2Bridge
  ) private view returns (uint value) {
      (address bridgeToken, LzBridgeData memory lzBridgeData) = abi.decode(
          additionalArgs,
          (address, LzBridgeData)
      );
      return
          bridgeToken == stargateEth
              ? (lzBridgeData.fee + amt2Bridge)
              : lzBridgeData.fee;
  }
```

## [L-02] 1-1 redemptions of WETH & ETH will fail for `DecentEthRouter`
The `DecentEthRouter::onlyIfWeHaveEnoughReserves()` modifier forces the caller to reduce their redemption amount by at least a WEI in order to fulfill redemptions of WETH or ETH as it makes sure the DecentEthRouter has more balance than the current amount being redeemed. So for every last user in an interval where the contract holds ETH or WETH relative to a 1:1 of their current amount to redeem, they will be forced to reduce their amount by 1 WEI or more.

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L50-L53

```solidity
  modifier onlyIfWeHaveEnoughReserves(uint256 amount) {
        require(weth.balanceOf(address(this)) > amount, "not enough reserves"); // @audit-low 1-1 redemptions will fail
        _;
    }
```

```solidity
function redeemWeth(
        uint256 amount
    ) public onlyIfWeHaveEnoughReserves(amount) { // @audit-issue LOW/INFO this will fail for the last user to redeem at any time.
    /*  
        1. suppose this contract has 100 weth
        2. Bob tries to redeem all 100 weth
        3. the `onlyIfWeHaveEnoughReserves` will make it fail because we can't redeem 1:1
        4. so Bob needs to reduce the amount they want to redeem to something like 999.99999 weth
    */
        dcntEth.transferFrom(msg.sender, address(this), amount);
        weth.transfer(msg.sender, amount);
    }
```

```solidity
function redeemEth(
        uint256 amount
    ) public onlyIfWeHaveEnoughReserves(amount) { 
        // @note same thing that affects redeemWeth, affects this
        dcntEth.transferFrom(msg.sender, address(this), amount);
        weth.withdraw(amount);
        payable(msg.sender).transfer(amount);
    }
```

An easy fix for this is add the = symbol.
```diff
modifier onlyIfWeHaveEnoughReserves(uint256 amount) {
  -  require(weth.balanceOf(address(this)) > amount, "not enough reserves");
    _;

  +  require(weth.balanceOf(address(this)) >= amount, "not enough reserves");
  _;
}
```

## [L-03] `UTBFeeCollector::collectFees` signature validation does not follow EIP-712

https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBFeeCollector.sol#L49-L54

The implementation of the signature verification in `UTBFeeCollector::collectFees` does not follow the `EIP-712` standard for Typed structured data hashing and signing, potentially leading to signature replay. 
```javascript
        bytes32 constructedHash = keccak256(
            abi.encodePacked(BANNER, keccak256(packedInfo)) // --> message encoding does not follow EIP-712
        );
        (bytes32 r, bytes32 s, uint8 v) = splitSignature(signature);
        address recovered = ecrecover(constructedHash, v, r, s);
```
Specifically, the absence of `domainSeparator` and `hashStruct`, along with the incorrect `BANNER` value (should be \x19\x01), may allow actors to re-use valid signatures to authorize transactions or actions repeatedly. Plus if external protocols are expected to work with signatures that conform to EIP-712, integration with such protocols may fail.

### Recommandation
Ensure that the signature verification process fully complies with [EIP-712](https://eips.ethereum.org/EIPS/eip-712), including the use of domainSeparator, hashStruct and the correct Banner value. The struct used in `packedInfo` consists of the concatenation of two structs `abi.encode(instructions, fees)` and their encoding should be constructed as exlained in the [section](https://eips.ethereum.org/EIPS/eip-712#definition-of-encodetype).

## [NC-01] Rename wrapped variable to wrappedNative for readibility
We understand that the `IWETH wrapped` variable in the `UTB.sol` contract refers to the interface of the native token for whatever chain the contract is deployed on e.g on Mainnet it will be WETH and on Polygon, it will be WMATIC so it makes more sense to switch the naming from `wrapped` to `wrappedNative`

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L20

```diff
- IWETH wrapped;
+ IWETH wrappedNative;
```