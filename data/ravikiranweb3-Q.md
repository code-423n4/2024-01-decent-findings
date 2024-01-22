#### 1) UTBFeeCollector::collectFees() 
validates the signature against the constructed hash from packedInfo. There is no nonce opening for replay attack. The layer of protection is only utb access and hence a low as best practices was not followed.


#### 2) StargateBridgeAdapter::bridge()
Destination chain id should be checked at the beginning of the function.But, instead, it is checked very late during the callBridge() function where if checks for the destinationBridgeAdapter for the destination chain id. If the address returned from the mapping is 0, the transaction reverts.

#### 3) DecentBridgeAdapter::bridgeToken 
variable can be declared as immutable to save gas.
#### 4) DecentBridgeAdapter::gasIsEth  
variable can be declared as immutable to save gas.

#### 5) DecentEthRouter::userDepositing && userIsWithdrawing modifiers
Modifiers should be used to validate data, enforce access control, but state variable update should be done only in functions. This is not the best practices.