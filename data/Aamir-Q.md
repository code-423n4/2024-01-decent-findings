# Summary

| Severity        | Info                                                                        | Instances |
| --------------- | --------------------------------------------------------------------------- | --------- |
| [[L-0](#low-0)] | Signature Can be replayed and a user can get away by paying less fee        | 1         |
| [[L-1](#low-1)] | All Approvals should be cleared in `DecentEthExector::_executeWeth()`       | 1         |
| [[L-2](#low-2)] | Add functionality to retryfailed Message from Layerzero                     | 1         |
| [[L-3](#low-3)] | Contracts that holds tokens should have emergency withdraw tokens functions | 2         |
| [[L-4](#low-4)] | There should be `deadline` for swap token calls                             | 1         |

# Lows

## L-0 Signature Can be replayed and a user can get away by paying less fee <a id="low-0"></a>

`UTBFeeCollector::collectFees(...)` uses ECDSA signatures in order to verify if the correct fee is paid by the user. And as per the sponsors the fee can be paid as a BPS of the amount swapped or bridged or in any other way and it is not fixed. So there are chances that it might be raised in the future. A user can exploit this behaviour. He can use an already used signature that had lower fees and can get away by paying the lower fee than current fee. All he has to do is use the same paramaters for the new call that he or someone else used before. This is exploitable because the signatures are not completely unique. It is a combination of the following things:

`BANNER` + `SwapAndExecuteInstructions` + `FeeStructure`

where,

`BANNER`:
`"\x19Ethereum Signed Message:\n32"`

`SwapAndExecuteInstructions`:

```solidity
struct SwapAndExecuteInstructions {
    SwapInstructions swapInstructions;
    address target;
    address paymentOperator;
    address payable refund;
    bytes payload;
}
```

`FeeStructure`:

```solidity
struct FeeStructure {
    uint256 bridgeFee;
    address feeToken;
    uint256 feeAmount;
}
```

The reason why the signature is not unique is because these all are user supplied inputs and if a user wants to make the same call in the future(with same parameter) all he has to do is pick up the old signature and use it. And it will still work. But this type of situation will only work if the following condition is met:

1. The fee BPS has increasd.
2. If it is not an external call that itself will be unique like, minting an NFT.

Not only that a user can pick up the signature from other chain that has same parameter with low fee and it will still work.

#### Mitigation

It is recommended to make the signature unique. Add something like an incremental nonce and chainId to the signature.

---

## L-1 All Approvals should be cleared at the end in `DecentEthExector::_executeWeth()` <a id="low-1"></a>

If the call fails in `DecentEthExector::_executeWeth()`, then the weth is transferred to the target address but the approval to `from` still exists. This can be exploited by the someone if the contract is under an attack or a tokens is used that will only allow approval if the old approval is not zero(in future ofcourse). Also there will be an active approval if all of tokens are not used by the target contract.

```solidity
    function _executeWeth(
        address from,
        address target,
        uint256 amount,
        bytes memory callPayload
    ) private {
        uint256 balanceBefore = weth.balanceOf(address(this));
        // @audit approval will remain
@>        weth.approve(target, amount);

        (bool success, ) = target.call(callPayload);

        if (!success) {
           weth.transfer(from, amount);
            return;
        }

        uint256 remainingAfterCall = amount -
            (balanceBefore - weth.balanceOf(address(this)));

        // refund the sender with excess WETH
        weth.transfer(from, remainingAfterCall);
    }
```

#### Mitigation

Clear any approvals at the end of the functions.

---

## L-2 Add functionality to retryfailed Message from Layerzero

If message is failed to be sent by the layerzero due to some reason, there is a functionality that can be used to retry the failed message. It can be called by anyone. But in order to do that someone has to go to the layerzero endpoint and pass in the correct parameters in `lzReceive` to retreive the Message. But there is no function in `DecentEthRouter` or any other contract to do that. This functionality should be added so that user does not have to go anywhere.

To learn more about it, go to this [link](https://docs.layerzero.network/contracts/debugging-messages#retry-message)

---

## L-3 Contracts that holds tokens should have emergency withdraw tokens functions <a id="low-3"></a>

Contracts that holds ETH or ERC20 should have emergencey withdraw tokens function. Atleast `DecentEthRouter` should have it. So that if the tokens are stucked in the contract due to any reason then it can be retreived from the contract. Or if any unexpected token is sent than it can be recoverd as well. These functions are also helpful in case the contract is under attack. But there is no such function. It is recommended to add these functions.

---

## L-4 There should be `deadline` for swap token calls <a id="low-4"></a>

Currently swap functions doesn't accept deadline parameter for swaps. This deadline ensures that the tokens are received within a fixed time period so that no unexpeted results comes from the swaps. But there is no such parameter used. It is recommeded to add the deadline as well
