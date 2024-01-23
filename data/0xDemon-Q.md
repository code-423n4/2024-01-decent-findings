Use of ecrecover is susceptible to signature malleability

The built-in EVM precompile `ecrecover` is susceptible to signature malleability, which could lead to replay attacks. References: https://swcregistry.io/docs/SWC-117, https://swcregistry.io/docs/SWC-121.

*There are 1 instance(s) of this issue:*

```solidity
File : src/UTBFeeCollector.sol

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

*Github : [44 - 62](https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBFeeCollector.sol#L44-L62)*