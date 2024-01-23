# [L-01] `UTBFeeCollector` is missing a check for `ecrecover` returning `address(0)` #

## Description ##

When [UTBFeeCollector](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L53) checks a given signature, it uses `ecrecover` to calculate the address in order to verify the fee structure.


    52    (bytes32 r, bytes32 s, uint8 v) = splitSignature(signature);
    53    address recovered = ecrecover(constructedHash, v, r, s);
    54    require(recovered == signer, "Wrong signature");

When provided an invalid signature, `ecrecover` may return `address(0)`. [This is possible because `uint8 v` is not validated.](https://github.com/kadenzipfel/smart-contract-vulnerabilities/blob/master/vulnerabilities/unexpected-ecrecover-null-address.md) This missing check could be exploited in an attack.

## Recommended Mitigation ##

Consider implementing an additional check for `recovered != address(0)` or add a check for `v` to be either 27, or 28.


# [N-01] Missing names/comments in function parameters in `DecentEthRouter` #

## Description ##

[DecentEthRouter](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L240) is using parameters without names or comments.

    237    function on onOFTReceived(
    238        uint16 _srcChainId,
    239        bytes calldata,
    240        uint64,
    241        bytes32,
    242        uint _amount,
    243        bytes memory _payload

## Recommended Mitigation ##

Consider adding names or NatSpec comments to the `uint64` and `bytes32` parameters. This will enhance the clarity of the code and make it easier to understand the purpose of each input.