# L001 - Lax signature check by UTBFeeCollector

## Impact

When the `UTBFeeCollector.signer` value is the zero address any invalid signature will be validated by `collectFees` due to not checking the signer address against the zero address.

```solidity
function collectFees(
    FeeStructure calldata fees,
    bytes memory packedInfo,
    bytes memory signature
) public payable onlyUtb {
    bytes32 constructedHash = keccak256(
        abi.encodePacked(BANNER, keccak256(packedInfo))
    );
    (bytes32 r, bytes32 s, uint8 v) = splitSignature(signature);
    address recovered = ecrecover(constructedHash, v, r, s);
    // @audit-issue `recovered` is not validated against the zero address
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

From: https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L44:L62

## Tools Used

Manual Review

## Recommended Mitigation Steps

Either use the latest [OpenZeppelin ECDSA cryptography library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/01ef448981be9d20ca85f2faf6ebdf591ce409f3/contracts/utils/cryptography/ECDSA.sol) (recommended, as it has been audited and not only checks that the signature is not malleable but also checks that the signer address is not the zero address) or make sure to implement the logic that validates address against the zero address in the `collectFees` function. For instance:

```solidity
function collectFees(
    FeeStructure calldata fees,
    bytes memory packedInfo,
    bytes memory signature
) public payable onlyUtb {
    bytes32 constructedHash = keccak256(
        abi.encodePacked(BANNER, keccak256(packedInfo))
    );
    (bytes32 r, bytes32 s, uint8 v) = splitSignature(signature);
    address recovered = ecrecover(constructedHash, v, r, s);
    // HERE
    require(recovered == address(0), "Wrong signature");
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
