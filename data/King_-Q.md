## Per Layerzero recommendations zroPaymentAddress should not be hardcoded as address(0x0) in the call params to allow for future ZRO fee payments and prevent Bridge Agents from falling apart in case LayerZero makes breaking changes

## Affected line
https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L172

## Recommendation
Consider passing zroPaymentAddress as an input parameter to allow future payments with Layerzero's native ZRO token which is speculated to launch sometime in 2024