

## L1 Lack of address(0) check of `target` and `refund`

https://github.com/code-423n4/2024-01-decent/blob/d562762c3bf58cca7e24171873fb7b6fbfa7b2b5/src/UTB.sol#L108-L124

`target` and `refund` lack of address(0) check, which may cause unexpected  fund loss.

```solidity
    function swapAndExecute(
        SwapAndExecuteInstructions calldata instructions,
        FeeStructure calldata fees,
        bytes calldata signature
    )
        public
        payable
        retrieveAndCollectFees(fees, abi.encode(instructions, fees), signature)
    {
    
        _swapAndExecute(
            instructions.swapInstructions,
            instructions.target,
            instructions.paymentOperator,
            instructions.payload,
            instructions.refund
        );
    }
```