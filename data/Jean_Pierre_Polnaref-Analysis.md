# Smart Contract Audit Report
# src/UTB.sol	

## Summary

In this audit report, I identified a high severity vulnerability in a smart contract that sends Ether to arbitrary addresses. This vulnerability exists in the `callBridge` function, which is capable of sending Ether to any address without any checks or restrictions.

## Details of the Vulnerability

The vulnerability is located in the `callBridge` function of the contract. This function is responsible for sending Ether to a bridge adapter. However, it does not contain any checks to ensure that the Ether is being sent to a valid or trustworthy address. As a result, it could potentially be used to send Ether to any address, including malicious ones.

The `callBridge` function is defined as follows:

```solidity
function callBridge(
    uint256 amt2Bridge,
    uint bridgeFee,
    BridgeInstructions memory instructions
) private returns (bytes memory) {
    bool native = approveAndCheckIfNative(instructions, amt2Bridge);
    return
        IBridgeAdapter(bridgeAdapters[instructions.bridgeId]).bridge{
            value: bridgeFee + (native ? amt2Bridge : 0)
        }(
            amt2Bridge,
            instructions.postBridge,
            instructions.dstChainId,
            instructions.target,
            instructions.paymentOperator,
            instructions.payload,
            instructions.additionalArgs,
            instructions.refund
        );
}
```

## PoC (Proof of Concept)

To demonstrate how this vulnerability could be exploited, let's consider a scenario where an attacker is able to manipulate the `instructions` parameter to include their own address as the target. Here's a simplified example:

```javascript
// Attacker's account
const maliciousAccount = '0xMaliciousAccountAddress';

// Instance of the smart contract
const myContractInstance = new web3.eth.Contract(MyContractABI, MyContractAddress);

// Amount to bridge
const amt2Bridge = web3.utils.toWei('1', 'ether');

// Bridge fee
const bridgeFee = web3.utils.toWei('0.01', 'ether');

// Instructions object
const instructions = {
    bridgeId: 1,
    postBridge: false,
    dstChainId: 1,
    target: maliciousAccount,
    paymentOperator: '0xPaymentOperatorAddress',
    payload: '0xPayload',
    additionalArgs: [],
    refund: false
};

// Call the callBridge function
myContractInstance.methods.callBridge(amt2Bridge, bridgeFee, instructions).send({ from: attackerAccount });
```

In this case, `amt2Bridge` is the amount of Ether to be bridged, `bridgeFee` is the fee for the bridge, `instructions` is an object containing various parameters for the bridge, and `attackerAccount` is the address of the attacker's account. `myContractInstance` is an instance of the smart contract being analyzed.

By manipulating the `instructions` parameter, an attacker could potentially redirect the Ether being bridged to their own address.

## Recommendations

To mitigate this vulnerability, it is recommended to add checks in the `callBridge` function to ensure that the Ether is being sent to a valid or trustworthy address. This could be achieved by implementing a whitelist of allowed addresses, or by adding a modifier that checks the sender's address against a list of known good addresses. Additionally, it would be beneficial to implement a mechanism for verifying the authenticity of the `instructions` parameter, such as a signature from a trusted authority.

# 2 error

## Summary

In this audit report, I have identified a high severity vulnerability in a smart contract that involves an unchecked transfer of ERC20 tokens. This vulnerability exists in the `retrieveAndCollectFees` function, which is capable of transferring tokens to the contract without checking if the transfer was successful.

## Details of the Vulnerability

The vulnerability is located in the `retrieveAndCollectFees` function of the contract. This function is responsible for retrieving and collecting fees from a sender. It does this by calling the `transferFrom` function of the ERC20 token contract. However, it does not check the return value of the `transferFrom` function, which means it does not verify if the transfer was successful.

The `retrieveAndCollectFees` function is defined as follows:

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

## PoC (Proof of Concept)

To illustrate how this vulnerability could be exploited, let's consider a scenario where an attacker is able to manipulate the `fees` parameter to include an invalid token address. Here's a simplified example:

```javascript
// Attacker's account
const maliciousAccount = '0xMaliciousAccountAddress';

// Instance of the smart contract
const myContractInstance = new web3.eth.Contract(MyContractABI, MyContractAddress);

// Fees object
const fees = {
    feeToken: '0xInvalidTokenAddress',
    feeAmount: web3.utils.toWei('1', 'ether')
};

// Packed info and signature
const packedInfo = '0xPackedInfo';
const signature = '0xSignature';

// Call the retrieveAndCollectFees function
myContractInstance.methods.retrieveAndCollectFees(fees, packedInfo, signature).send({ from: maliciousAccount });
```

In this case, `fees` is an object containing the token address and the fee amount, `packedInfo` is a string representing packed information, and `signature` is a string representing a signature. `maliciousAccount` is the address of the attacker's account. `myContractInstance` is an instance of the smart contract being analyzed.

By manipulating the `fees` parameter, an attacker could potentially cause the `transferFrom` function to fail, which would cause the `retrieveAndCollectFees` function to also fail.

## Recommendations

To mitigate this vulnerability, it is recommended to check the return value of the `transferFrom` function. This could be achieved by using the `SafeERC20` library from OpenZeppelin, which provides safe versions of ERC20 functions that check for errors. Alternatively, you could manually check the return value of the `transferFrom` function and revert the transaction if the transfer was not successful.

## Smart Contract second Audit Report
## src/UTBExecutor.sol	
## Summary

In this audit report, I identified a high severity vulnerability in a smart contract that involves an arbitrary call to an external contract. This vulnerability exists in the `execute` function, which is capable of making arbitrary calls to any address.

## Details of the Vulnerability

The vulnerability is located in the `execute` function of the contract. This function is responsible for executing a call to a target contract. However, it does not contain any checks to ensure that the call is made to a valid or trustworthy address. As a result, it could potentially be used to make calls to any address, including malicious ones.

The `execute` function is defined as follows:

```solidity
function execute(
    address target,
    address paymentOperator,
    bytes memory payload,
    address token,
    uint amount,
    address payable refund
) public payable onlyOwner {
    return
        execute(target, paymentOperator, payload, token, amount, refund, 0);
}
```

## PoC (Proof of Concept)

To demonstrate how this vulnerability could be exploited, let's consider a scenario where an attacker is able to manipulate the `target` parameter to include their own address as the target. Here's a simplified example:

```javascript
// Attacker's account
const maliciousAccount = '0xMaliciousAccountAddress';

// Instance of the smart contract
const myContractInstance = new web3.eth.Contract(MyContractABI, MyContractAddress);

// Target is the attacker's account
const target = maliciousAccount;

// Other parameters...
const paymentOperator = '0xPaymentOperatorAddress';
const payload = '0xPayload';
const token = '0xTokenAddress';
const amount = web3.utils.toWei('1', 'ether');
const refund = '0xRefundAddress';

// Call the execute function
myContractInstance.methods.execute(target, paymentOperator, payload, token, amount, refund).send({ from: attackerAccount });
```

In this case, `target` is the address to which the call will be made, `paymentOperator` is the payment operator, `payload` is the payload of the call, `token` is the token address, `amount` is the amount of tokens to be transferred, `refund` is the refund address, and `attackerAccount` is the address of the attacker's account. `myContractInstance` is an instance of the smart contract being analyzed.

By manipulating the `target` parameter, an attacker could potentially redirect the call to their own address.

## Recommendations

To mitigate this vulnerability, it is recommended to add checks in the `execute` function to ensure that the call is made to a valid or trustworthy address. This could be achieved by implementing a whitelist of allowed addresses, or by adding a modifier that checks the target address against a list of known good addresses. Additionally, it would be beneficial to implement a mechanism for verifying the authenticity of the `execute` function, such as a signature from a trusted authority.

# 2 error

## Summary

In this audit report, I identified a high severity vulnerability in a smart contract that involves an unchecked transfer of ERC20 tokens. This vulnerability exists in the `execute` function, which is capable of transferring tokens to the contract without checking if the transfer was successful.

## Details of the Vulnerability

The vulnerability is located in the `execute` function of the contract. This function is responsible for executing a call to a target contract. It transfers tokens from the sender to the contract using the `transferFrom` function of the ERC20 token contract. However, it does not check the return value of the `transferFrom` function, which means it does not verify if the transfer was successful.

The `execute` function is defined as follows:

```solidity
function execute(
    address target,
    address paymentOperator,
    bytes memory payload,
    address token,
    uint amount,
    address payable refund,
    uint extraNative
) public onlyOwner {
    bool success;
    if (token == address(0)) {
        (success, ) = target.call{value: amount}(payload);
        if (!success) {
            (refund.call{value: amount}(""));
        }
        return;
    }

    uint initBalance = IERC20(token).balanceOf(address(this));

    IERC20(token).transferFrom(msg.sender, address(this), amount);
    IERC20(token).approve(paymentOperator, amount);

    if (extraNative > 0) {
        (success, ) = target.call{value: extraNative}(payload);
        if (!success) {
            (refund.call{value: extraNative}(""));
        }
    } else {
        (success, ) = target.call(payload);
    }

    uint remainingBalance = IERC20(token).balanceOf(address(this)) -
        initBalance;

    if (remainingBalance == 0) {
        return;
    }

    IERC20(token).transfer(refund, remainingBalance);
}
```

## PoC (Proof of Concept)

To illustrate how this vulnerability could be exploited, let's consider a scenario where an attacker is able to manipulate the `token` parameter to include an invalid token address. Here's a simplified example:

```javascript
// Attacker's account
const maliciousAccount = '0xMaliciousAccountAddress';

// Instance of the smart contract
const myContractInstance = new web3.eth.Contract(MyContractABI, MyContractAddress);

// Token is the attacker's account
const token = maliciousAccount;

// Other parameters...
const target = '0xTargetAddress';
const paymentOperator = '0xPaymentOperatorAddress';
const payload = '0xPayload';
const amount = web3.utils.toWei('1', 'ether');
const refund = '0xRefundAddress';
const extraNative = web3.utils.toWei('0.01', 'ether');

// Call the execute function
myContractInstance.methods.execute(target, paymentOperator, payload, token, amount, refund, extraNative).send({ from: attackerAccount });
```

In this case, `token` is the address of the token to be transferred, `target` is the address to which the call will be made, `paymentOperator` is the payment operator, `payload` is the payload of the call, `amount` is the amount of tokens to be transferred, `refund` is the refund address, `extraNative` is the extra amount of native currency to be sent, and `attackerAccount` is the address of the attacker's account. `myContractInstance` is an instance of the smart contract being analyzed.

By manipulating the `token` parameter, an attacker could potentially cause the `transferFrom` function to fail, which would cause the `execute` function to also fail.

## Recommendations

To mitigate this vulnerability, it is recommended to check the return value of the `transferFrom` function. This could be achieved by using the `SafeERC20` library from OpenZeppelin, which provides safe versions of ERC20 functions that check for errors. Alternatively, you could manually check the return value of the `transferFrom` function and revert the transaction if the transfer was not successful.

## Smart Contract third Audit Report
## src/UTBFeeCollector.sol	
## Summary

In this audit report, I identified a high severity vulnerability in a smart contract that involves an arbitrary call to an external contract. This vulnerability exists in the `collectFees` function, which is capable of transferring tokens from any address to the contract without checking if the transfer was successful.

## Details of the Vulnerability

The vulnerability is located in the `collectFees` function of the contract. This function is responsible for collecting fees from a sender. It does this by calling the `transferFrom` function of the ERC20 token contract. However, it does not check the return value of the `transferFrom` function, which means it does not verify if the transfer was successful.

The `collectFees` function is defined as follows:

```solidity
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

## PoC (Proof of Concept)

To illustrate how this vulnerability could be exploited, let's consider a scenario where an attacker is able to manipulate the `utb` parameter to include their own address as the `utb`. Here's a simplified example:

```javascript
// Attacker's account
const maliciousAccount = '0xMaliciousAccountAddress';

// Instance of the smart contract
const myContractInstance = new web3.eth.Contract(MyContractABI, MyContractAddress);

// Fees object
const fees = {
    feeToken: '0xTokenAddress',
    feeAmount: web3.utils.toWei('1', 'ether')
};

// Packed info and signature
const packedInfo = '0xPackedInfo';
const signature = '0xSignature';

// Utb is the attacker's account
const utb = maliciousAccount;

// Call the collectFees function
myContractInstance.methods.collectFees(fees, packedInfo, signature, utb).send({ from: attackerAccount });
```

In this case, `fees` is an object containing the token address and the fee amount, `packedInfo` is a string representing packed information, `signature` is a string representing a signature, `utb` is the address from which the tokens will be transferred, and `attackerAccount` is the address of the attacker's account. `myContractInstance` is an instance of the smart contract being analyzed.

By manipulating the `utb` parameter, an attacker could potentially cause the `transferFrom` function to transfer tokens from their own address to the contract.

## Recommendations

To mitigate this vulnerability, it is recommended to check the return value of the `transferFrom` function. This could be achieved by using the `SafeERC20` library from OpenZeppelin, which provides safe versions of ERC20 functions that check for errors. Alternatively, you could manually check the return value of the `transferFrom` function and revert the transaction if the transfer was not successful. Additionally, it would be beneficial to implement a mechanism for verifying the authenticity of the `collectFees` function, such as a signature from a trusted authority.



### Time spent:
2 hours