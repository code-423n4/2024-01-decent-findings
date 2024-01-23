## [L-01] `msg.value` is not correctly handled

In `performSwap`, the following requirement is set:
```javascript
require(msg.value >= swapParams.amountIn, "not enough native");
```

This requirement checks if the sender has sent enough `msg.value` for a `NATIVE` to `NON-NATIVE` swap.

However, this require statement does not take into account the additional `msg.value` that the user needs down the line, for example, if `msg.value == swapParams.amountIn` and the `msg.value` will be turned into `tokenOut, amountOut`, then there will be a problem for example here:
```javascript
    function callBridge(
        uint256 amt2Bridge,
        uint bridgeFee,
        BridgeInstructions memory instructions
    ) private returns (bytes memory) {
        bool native = approveAndCheckIfNative(instructions, amt2Bridge);
        return
            IBridgeAdapter(bridgeAdapters[instructions.bridgeId]).bridge{
                value: bridgeFee + (native ? amt2Bridge : 0)
            }(
                amt2Bridge,
                instructions.postBridge,
                instructions.dstChainId,
                instructions.target,
                instructions.paymentOperator,
                instructions.payload,
                instructions.additionalArgs,
                instructions.refund
            );
    }
```

After the swaps happened inside `performSwap`, we will eventually end up here. Note that the call `.bridge` is accompanied with `value: bridgeFee + (native ? amt2bridge : 0)`.
If the whole `msg.value` has been transferred to `tokenOut, amountOut` then this will fail due to the fact that `msg.value` will be `0`, meaning there will not be enough `bridgeFee` to complete the transaction.

**Recommendation**: In the require statement, handle the amount that the user needs to have in `msg.value` to complete the `cross-chain transaction`, not just the `performSwap` operation.

-----------------
## [L-02] Hardcoded gas values for relayers

Hard-coded gas parameters can lead to failures on different blockchains.

In `DecentEthRouter_getcallParams`, the value `GAS_FOR_RELAY` is hard-coded:
```javascript
uint256 GAS_FOR_RELAY = 100000;
```

However, different networks have different block size limits, gas limits.

**Recommendation**: do not use a hard-coded value, use getters provided by `LayerZero`.

---------------------------------------------------------------------
## [L-03] Signature can be replayed on different chains

Signatures can be replayed on different block chains due to the fact that this value is hardcoded:
```javascript
    string constant BANNER = "\x19Ethereum Signed Message:\n32";
```

This value is used inside of `collectFees`:
```javascript
    /**
     * @dev Receives fees in either native or ERC20.
     * @param fees The bridge fee in native, as well as UTB fee token and amount.
     * @param packedInfo The fees and swap instructions used to generate the signature.
     * @param signature The ECDSA signature to verify the fee structure.
     */
    function collectFees(
        FeeStructure calldata fees,
        bytes memory packedInfo,
        bytes memory signature
    ) public payable onlyUtb {
@>      bytes32 constructedHash = keccak256(
            abi.encodePacked(BANNER, keccak256(packedInfo))
        );
        (bytes32 r, bytes32 s, uint8 v) = splitSignature(signature);
        address recovered = ecrecover(constructedHash, v, r, s);
        require(recovered == signer, "Wrong signature");
        if (fees.feeToken != address(0)) {
            IERC20(fees.feeToken).transferFrom(
                utb,
                address(this),
                fees.feeAmount
            );
        }
	}
```
This means that a valid signature hashed using the same `BANNER` and `packedInfo`, can be replayed on different EVM's.

**Recommendation**: Since the Sponsor plans on using these contracts on different blockchains, change the `BANNER` to a unique value on each blockchain.

------------
## [L-04] CEI-pattern not followed inside of `modifier userDepositing`

Inside `modifier userDepositing`, the CEI-pattern is not correctly adhered to:
```javascript
    modifier userIsWithdrawing(uint256 amount) {
        uint256 balance = balanceOf[msg.sender];
        require(balance >= amount, "not enough balance");
        _;
        //@audit low, no ceipattern
        balanceOf[msg.sender] -= amount;
    }
```

The external calls are made before the balance is changed:

**Recommendation**: change the modifier to this:
```javascript
    modifier userIsWithdrawing(uint256 amount) {
        uint256 balance = balanceOf[msg.sender];
        require(balance >= amount, "not enough balance");
		balanceOf[msg.sender] -= amount;
        _;
    }

```

------
## [L-05] Do not hard-code `zroPaymentAddress`
As per [LayerZero Documentation](https://layerzero.gitbook.io/docs/troubleshooting/layerzero-integration-checklist) it is not recommended to hard-code the `zroPaymentAddress`:
```javascript
file: DecentEthRouter.sol
        ICommonOFT.LzCallParams memory callParams = ICommonOFT.LzCallParams({
            refundAddress: payable(msg.sender),
            zroPaymentAddress: address(0x0),
            adapterParams: adapterParams
        });
```

**Recommendation**:
```diff
        ICommonOFT.LzCallParams memory callParams = ICommonOFT.LzCallParams({
            refundAddress: payable(msg.sender),
-            zroPaymentAddress: address(0x0),
+            zroPaymentAddress: someInputParam,
            adapterParams: adapterParams
        });
```

----
## [L-06] Zero value from 'ecrecover' not checked

In `UTBFeeCollector.collectFees`, `ecreceover()` gets called:
```javascript
    /**
     * @dev Receives fees in either native or ERC20.
     * @param fees The bridge fee in native, as well as UTB fee token and amount.
     * @param packedInfo The fees and swap instructions used to generate the signature.
     * @param signature The ECDSA signature to verify the fee structure.
     */
    function collectFees(
        FeeStructure calldata fees,
        bytes memory packedInfo,
        bytes memory signature
    ) public payable onlyUtb {
        bytes32 constructedHash = keccak256(
            abi.encodePacked(BANNER, keccak256(packedInfo))
        );
        (bytes32 r, bytes32 s, uint8 v) = splitSignature(signature);
@>      address recovered = ecrecover(constructedHash, v, r, s);
        require(recovered == signer, "Wrong signature");
        if (fees.feeToken != address(0)) {
            IERC20(fees.feeToken).transferFrom(
                utb,
                address(this),
                fees.feeAmount
            );
        }
    }
```

The return address gets put into the `recovered` variable and then `recovered`  gets checked against `signer`.

However, there is no `0` check. This is problematic because, if `signer` is not set to an address, any invalid signature is able to pass  `require(recovered == signer, "Wrong signature")`.

Since `collectFees` is part of the `modifier retrieveAndCollectFees`:
```javascript
modifier retrieveAndCollectFees(
        FeeStructure calldata fees,
        bytes memory packedInfo,
        bytes calldata signature
    ) {
        if (address(feeCollector) != address(0)) {
            uint value = 0;
            if (fees.feeToken != address(0)) {
                IERC20(fees.feeToken).transferFrom(
                    msg.sender,
                    address(this),
                    fees.feeAmount
                );
                IERC20(fees.feeToken).approve(
                    address(feeCollector),
                    fees.feeAmount
                );
            } else {
                value = fees.feeAmount;
            }
            feeCollector.collectFees{value: value}(fees, packedInfo, signature);
        }
        _;
    }
```

Anyone will be use `bridgeAndExecute` and `swapAndExecute` with invalid fee data, circumventing the user fee payments.

**Recommendation**: Add a `0` check inside of `collectFees`.
