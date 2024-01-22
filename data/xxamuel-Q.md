https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTB.sol#L83-L90


 Use IERC20 for Transfer: Instead of creating a new instance of IERC20 for every transfer, you can store the token contract instance outside the function if it's used multiple times within the contract.
