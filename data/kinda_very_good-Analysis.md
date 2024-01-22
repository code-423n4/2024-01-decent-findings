### [H-1] UTB::receiveFromBridge allows users to swap without fees 

**Description:** 
The UTB::receiveFromBridge does not implement the UTB::retrieveAndCollectFees modifier which ensures fees were signed by the user prompting users to call it instead of UTB::swapAndExecute

**Impact:** The protocol could lose revenue from fees

**Proof of Concept:**
```
  function receiveFromBridge(
        SwapInstructions memory postBridge,
        address target,
        address paymentOperator,
        bytes memory payload,
        address payable refund
    ) public {
        _swapAndExecute(postBridge, target, paymentOperator, payload, refund);
    }
```

**Recommended Mitigation:** UTB::receiveFromBridge should be have proper visibility to ensure it can only be called from inside the function





### [M-3] UniSwapper::swapNoPath does not account for tokens of different decimals

**Description:** 

**Impact:** User could end up being refunded more than the value of the out token


**Proof of Concept:**

```
 if (swapParams.direction == SwapDirection.EXACT_OUT) {
            _refundUser(
                refund,
                swapParams.tokenIn,
                swapParams.amountIn - swapParams.amountOut // tokens of different decimals ?
            );
        }
```
The block of code above refunds the user the result of the (amount in - amount out) , this could cause an error if the amount in token is greater than the amount out e.g 18 decimals and 6 decimals, the user would end up being refunded a significant amount of their tokens 

**Recommended Mitigation:** 










### [H-2] UniSwapper::swap functions do not check if the reciever or the refund is address(0)

**Description:** The UniSwapper::_refund and UniSwapper::_sendToRecipient does not chekc if the reciever or refund is address(0) before transferring tokens , for tokens that do not revert on zero address transfer , this could lead to tokens being lost

**Impact:** Users funds could be lost 

**Recommended Mitigation:** A zero address check should be enforced when transferring tokens 









### [H-3] UTB::PerformSwap functions do not ensure that the tokenOut is same as the last address in path

**Description:** The swapParams out has a seperate slot for the address of the outTokens, when a swap is executed there are no actual checks that the token out is the same as the out token encoded in path, this could lead to the system believing a different token was swapped than what was actually swapped

**Impact:** Wrong information could be passed to the bridge 


**Recommended Mitigation:** The token In and token out should be set to the first and last twenty bytes of the paths bytes array to ensure accuracy



### [H-3] UTB::PerformSwap calls swap with the initial swap instructions but does not actually send the value when the token in is address(0) (eth)

**Description:** UTB::PerformSwap does not send value to the swap call but still send the initial instructions of the token in being address zero meaning when UniSwapper::_receiveAndWrapIfNeeded is called 0 weth will be minted and used for swap 

**Impact:** All payloads with ether as token in transfers will fail when the router is called as swap

**Proof of Concept:**
```
(tokenOut, amountOut) = swapper.swap(swapInstructions.swapPayload); calls with initial instructions 

uniswapper
    function _receiveAndWrapIfNeeded(
        SwapParams memory swapParams
    ) private returns (SwapParams memory _swapParams) {
        if (swapParams.tokenIn != address(0)) {
            IERC20(swapParams.tokenIn).transferFrom(
                msg.sender,
                address(this),
                swapParams.amountIn
            );
            return swapParams;
        }
        swapParams.tokenIn = wrapped;
        IWETH(wrapped).deposit{value: swapParams.amountIn}();
        return swapParams;
    } // will mint zero weth

```

**Recommended Mitigation:** the swap function should be called with the updated swap params 







### [M-1] UTBFeeCollector::collectFees allows for reusable signatures 

**Description:** The UTBFeeCollector does not provide sufficient checks to ensure that the digest is accurate, when the signtaure signs a message the signature is efficient forever unless the signer is changed, this could lead to a problem if the digest is no longer meant to be correct

**Impact:** signatures that were valid when a payload was valid would still be valid when a payload is no longer valid 

```
A user could make a transaction , get a valid signature and execute the swap 
Assuming the preious fee or swapper id have now been invalidated , the user would still be able to execute the tx as long as they have the exact same data as the first one 
```
**Recommended Mitigation:** The signature should include the msg.sender and the nanace 







### [M-2] UTB::retrieveAndCollectFees does not check if the msg.value

**Description:** The UTB::retrieveAndCollectFees modifer sets the value if the feeToken is address(0) but doesnt actually check if the value was sent by the user 

**Impact:** Fees could end up not being sent

**Recommended Mitigation:** There should be a check to ensure that the value sent is greater or equal the token in and fee (assuming there are bot mant to be eth)





### [M-3] UTB::registerSwapper AND UTB::registerBridge could override the previous swapper or bridge if they have the same id

**Description:** The UTB::registerSwapper AND UTB::registerBridge do not check if those ids are already registered before setting a state variable 

**Impact:** Users could be deceived of the swapper and bridge executing their tx 


**Recommended Mitigation:** The UTB::registerSwapper AND UTB::registerBridge functions should hae logic to ensure that the ids are not already registered 





### [L-1] UTB does not have any fee set logic 

**Description:** There is no fee set in the utb code itself meaning all the fee logic is happening of chain

**Impact:** Users might not have accurate information of the fee 

**Recommended Mitigation:** The fee logic should be implemented on chain






### [L-3] setWrapped weth contract prolly does not need to be changed 

**Description:** The weth contract is not upgradeable so there is no need to update the weth contract , this could be a  means for protocol admins to set a false weth contract and invalidate the contract

**Impact:** The protocol could be invalidated for non erc to erc transfers

**Recommended Mitigation:** the weth contract should be immutable or constant




### [I-1]There is no situation where UTB::performSwap is set to false 

**Description:** Both UTB::performSwap functions effectively do the same thing 

**Impact:** Unnecessary usage of gas 

**Recommended Mitigation:** The perfromSwaps functions houdl be limited to only one 




### Time spent:
18 hours