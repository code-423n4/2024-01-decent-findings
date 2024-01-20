# QA Report

# Lack of `deadline` param in `UniSwapper`

## Lines 
https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/swappers/UniSwapper.sol#L129-L135

https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/swappers/UniSwapper.sol#L149-L156

## Description
The `UniSwapper` lacks a `deadline` parameter, which may lead to potential fund losses. Since front-running is an integral risk in automated market maker (AMM) platforms, setting a deadline is essential to prevent the possibility of a transaction being deliberately withheld and executed at an unfavorable time. Without this temporal constraint, validators might have an incentive to delay processing the transaction until the slippage reaches its peak, thereby extracting maximum profit to the detriment of the user's transaction.

A validator might postpone the transaction until market conditions, such as slippage, become favorable or execute their own transaction first, effectively front-running the original user's transaction for potential gain.

```solidity=129
        IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
            .ExactInputParams({
                path: swapParams.path,
                recipient: address(this),
                amountIn: swapParams.amountIn,
                amountOutMinimum: swapParams.amountOut
            });
```

```solidity=149
        IV3SwapRouter.ExactOutputParams memory params = IV3SwapRouter
            .ExactOutputParams({
                path: swapParams.path,
                recipient: address(this),
                //deadline: block.timestamp,
                amountOut: swapParams.amountOut,
                amountInMaximum: swapParams.amountIn
            });
```

## Recommendation
It's recommended to add a proper `deadline` param according to [parameter-structs](https://docs.uniswap.org/contracts/v3/reference/periphery/interfaces/ISwapRouter#parameter-structs).

# Missing Non-Zero Address Check for Return Value of `ecrecover()`

## Lines
https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBFeeCollector.sol#L53C1-L54C57

## Description
The `ecrecover` function in Solidity is instrumental in retrieving the address that signed a particular hash with an elliptic curve signature, defaulting to the address zero if the signature is invalid. This opcode is employed to extract a hash and a signature's originating address.

Within the `UTBFeeCollector` smart contract, its `collectFees` function invokes `ecrecover` to certify that the `packedInfo` was endorsed by an authorized `signer`. Before this verification, the `packedInfo` hash is prefixed with a standardized `BANNER` string, aligning with Ethereum's signed message format, and then supplied to `ecrecover` with the signature's segments `v`, `r`, and `s`.

Nonetheless, the contract omits a direct verification for the scenario where `ecrecover` may yield the zero address, a clear indication of a faulty signature. In such an event where the signature is erroneous, and `ecrecover` outputs the zero address, the contract lacks a mechanism to ensure the `signer` is assigned a legitimate value, which may lead to the subsequent `require` statement incorrectly succeeding.

Here is the pertinent excerpt from the code:

```solidity
        address recovered = ecrecover(constructedHash, v, r, s);
        require(recovered == signer, "Wrong signature");
```

In the current contract code, if `ecrecover` fails due to an invalid signature, it would return the zero address. If the `signer` variable is also uninitialized or purposely set to zero, the `require` check will not revert the transaction, and this could lead to unintended authorization of fee collection. This highlights a potential vulnerability in the contract where invalid signatures could pass through without proper verification.

## Recommendation
It is advisable to implement a check ensuring that the `ecrecover()` function does not return a zero address before proceeding with operations dependent on its outcome.