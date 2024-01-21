# [G-01] Use string literal instead of string concatenation
## Description
In the `DecentBridgeAdapter` smart contract, the `require` statement in the `bridge` function uses `string.concat`, which is unnecessary and increases gas costs.

## Recommended Mitigation Steps
Use a simple string literal.

GitHub: https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/bridge_adapters/DecentBridgeAdapter.sol#L93