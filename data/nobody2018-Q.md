# L-01: UTBFeeCollector.collectFees should check if recovered is 0

If [ecrecover(constructedHash, v, r, s)](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L53-L54) fails, it will return address(0x0), because the signer is not set when the UTBFeeCollector is created. Before the owner calls [setSigner](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L18-L20), the signer is equal to 0. In this way, the user can use an invalid signature to pay 0 fee.

## Recommended Mitigation Steps

```fix
File: src\UTBFeeCollector.sol
53:         address recovered = ecrecover(constructedHash, v, r, s);
54:-        require(recovered == signer, "Wrong signature");
54:+        require(recovered != address(0x0) && recovered == signer, "Wrong signature");
```

# L-02: The signed data should contain deadline and nonce

[swapAndExecute](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L108-L112)/[bridgeAndExecute](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L259-L263) all have a `signature` parameter, which is used to verify the fee structure. But the signature does not include the `deadline` and the caller's `nonces`. This is not best practice. Because the fee comes from off-chain (front-end data), if the fee is increased, users can use the old signature to perform operations. Therefore, the signature contains swap information, **but if it is a swapNoPath**, then using such a signature again will not have any impact on the user. In this way, the protocol will get less fees.

## Recommended Mitigation Steps

`signature` includes the `deadline` and the caller's `nonces`.

# L-03: UTBExecutor.execute will not work properly in some cases

UTBExecutor contract has two `execute` functions: [first](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L19), [second](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L41). The last parameter of second is `extraNative`, which is forwards additional gas or native fees required to execute the payment transaction.

If the owner calls first `execute` directly, which calls second `execute` internally, setting the `extraNative` parameter to 0. This is no problem.

If the owner wants to call second `execute` directly with `token != 0`, and set `extraNative` greater than 0. **This is impossible because it have no payable**.

```solidity
File: src\UTBExecutor.sol
41:     function execute(
42:         address target,
43:         address paymentOperator,
44:         bytes memory payload,
45:         address token,
46:         uint amount,
47:         address payable refund,
48:         uint extraNative
49:     ) public onlyOwner {//@@@@@@ this function has no payable
......
59:         uint initBalance = IERC20(token).balanceOf(address(this));
60: 
61:         IERC20(token).transferFrom(msg.sender, address(this), amount);
62:         IERC20(token).approve(paymentOperator, amount);
63: 
64:         if (extraNative > 0) {
65:->           (success, ) = target.call{value: extraNative}(payload);
66:             if (!success) {
67:                 (refund.call{value: extraNative}(""));
68:             }
69:         } else {
70:             (success, ) = target.call(payload);
71:         }
......
81:     }
```

## Recommended Mitigation Steps

Reconstruct the logic of second execute into an internal function, and then let both first `execute` and second `execute` call this internal function. And add `payable` modifier to second `execute`.

# L-04：In DecentBridgeExecutor._executeWeth, if target.call fails, the weth approval of the target should be revoked

```solidity
File: lib\decent-bridge\src\DecentBridgeExecutor.sol
24:     function _executeWeth(
25:         address from,
26:         address target,
27:         uint256 amount,
28:         bytes memory callPayload
29:     ) private {
30:         uint256 balanceBefore = weth.balanceOf(address(this));
31:->       weth.approve(target, amount);
32: 
33:         (bool success, ) = target.call(callPayload);
34: 
35:         if (!success) {
36:->           weth.transfer(from, amount);
37:             return;
38:         }
......
45:     }
```

Before `target.call(callPayload)` is called, `weth.approve(target, amount)` is called. Then, if `target.call` fails, it means that approve has not been spent. The target still has the allowance of this contract on WETH. If this contract holds WETH, then the target will be able to steal it.

## Recommended Mitigation Steps

```fix
35:         if (!success) {
36:             weth.transfer(from, amount);
+++		weth.approve(target, 0);
37:             return;
38:         }
```

# L-05: DecentEthRouter.onOFTReceived reverts in some cases

```solidity
File: lib\decent-bridge\src\DecentEthRouter.sol
237:     function onOFTReceived(
238:         uint16 _srcChainId,
239:         bytes calldata,
240:         uint64,
241:         bytes32,
242:         uint _amount,
243:         bytes memory _payload
244:     ) external override onlyLzApp {
......
266:         if (weth.balanceOf(address(this)) < _amount) {
267:->           dcntEth.transfer(_to, _amount);
268:             return;
269:         }
......
282:     }
```

If the amount of weth held by the DecentEthRouter contract is less than `_amount`, `dcntEth.transfer(_to, _amount)` will be called. This is equivalent to `1e18 weth =1e18 dcntEth`. However, `dcntEth.transfer` also transfers the dcntEth held by the DecentEthRouter contract to `_to`. **If the dcntEth held is less than `_amount`, then tx will be revert due to insufficient balance**.

This can happen. The whale can deposit WETH to `this` via [addLiquidityEth](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L302)/[addLiquidityWeth](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L322), and mint the same amount of dcntEth to `this`. Similarly,  he may also remove the amount of WETH and dcntEth of `this` contract via `removeLiquidityEth`/`removeLiquidityWeth`.  
When a large amount cross-chain transfer-message is sent from other chain, the whale can notice the tx, then remove liquidity on the target chain, then `onOFTReceived` will revert.

## Recommended Mitigation Steps

# L-06: UTB.receiveFromBridge should add access control

```solidity
File: src\UTB.sol
311:     function receiveFromBridge(
312:         SwapInstructions memory postBridge,
313:         address target,
314:         address paymentOperator,
315:         bytes memory payload,
316:         address payable refund
317:     ) public {
318:         _swapAndExecute(postBridge, target, paymentOperator, payload, refund);
319:     }
```

From function's name, `receiveFromBridge` should only be called by bridge, and we can also see it being called in [[1](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L147),[2](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L209)].

## Recommended Mitigation Steps

```fix
File: src\UTB.sol
311:     function receiveFromBridge(
312:         SwapInstructions memory postBridge,
313:         address target,
314:         address paymentOperator,
315:         bytes memory payload,
316:         address payable refund
317:     ) public {
+++	     require(bridgeAdapters[IBridgeAdapter(msg.sender).getId()] != address(0x0), "invalid caller");
318:         _swapAndExecute(postBridge, target, paymentOperator, payload, refund);
319:     }
```

&nbsp;