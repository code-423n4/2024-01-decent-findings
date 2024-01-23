# Findings Summary

| ID            | Title                                                                                                                                | Severity |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------ | -------- |
| [L-01](#l-01) | SwapperId Collision | Low      |
| [L-02](#l-02) | `UTB.retrieveAndCollectFees` lacks check if the funds received match the amount when its in ETH               | Low      |
| [L-03](#l-03) | User can indroduce invalid tokenOut               | Low      |
| [L-04](#l-04) | Sending more ETH than needed can result in locked funds               | Low      |

# Low

## [L-01] SwapperId Collision <a id="l-01"></a>

## Details

- When registering a new swapper in the `UTB.registerSwapper`, the swapperId is fetched from the swapper
allowing to overwrite the address of a swapperId.

## Proof of Concept

```solidity
function registerSwapper(address swapper) public onlyOwner {
        ISwapper s = ISwapper(swapper);
        swappers[s.getId()] = swapper;
    }
```

## Recommended Mitigation Steps

Generalize the first swapperId so that it cannot be overriden. Also a good practice would be to implement a deleting functionality.

## [L-02] `UTB.retrieveAndCollectFees` lacks check if the funds received match the amount when its in ETH <a id="l-02" ></a>

## Details

- In `UTB.retrieveAndCollectFees` there is a check if the ETH received matches the amount.

```solidity
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


## Proof of Concept

## Recommended Mitigation Steps

Implement a check if the ETH received matches the amount.

## [L-03] User can indroduce invalid tokenOut <a id="l-03" ></a>

## Details

- In the `Uniswapper.Swap()` function the `tokenOut` of the params and the path could not match.
The last address in `path` array should match `tokenOut`
## Proof of Concept

## Recommended Mitigation Steps

Implement a check if the last address in `path` matches the `tokenOut` address.

## [L-04] Sending more ETH than needed can result in locked funds <a id="l-04" ></a>

## Details

- When calling `UTB.SwapAndExecute()` there is a check

```solidity
require(msg.value >= swapParams.amountIn, "not enough native");
```
which checks if the ETH sent with the transaction is enough, but is it is more than the needed, it will be stuck into the contract.
## Proof of Concept

## Recommended Mitigation Steps

Change the check as follows:
```solidity
require(msg.value == swapParams.amountIn, "not enough native");
```