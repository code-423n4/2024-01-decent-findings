## [L-1] The `bool success` returned value  of address.call not validated

There is 1 instance of this
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L70

The boolean return value of `target.call(payload)` on line `71` of UTBExecutor.sol#execute(...) function was assigned but not validated.

```
File: UTBExecutor.sol
41: function execute(
        ...//Some lines omitted for clarity
49:    ) public onlyOwner {
...//Some lines omitted for clarity


64: if (extraNative > 0) {
65:            (success, ) = target.call{value: extraNative}(payload);
66:            if (!success) {
67:                (refund.call{value: extraNative}(""));
68:            }
69:        } else {
70:@>              (success, ) = target.call(payload);//@audit no success check.
71:        }
72:
73:        uint remainingBalance = IERC20(token).balanceOf(address(this)) -
74:            initBalance;


...//Some lines omitted for clarity

```

Impact: 
In situations where `target.call(payload)` fail, the transaction continues execution. This could lead to critical issues depending on the `target` address and the `payload`. It important to revert early when one of the intended actions fails.


Recommendation: 
Validate all return values of an `address.call` using:
```diff
     (success, ) = target.call(payload);
++   if(!success) revert("address.call failed"); 
```

## [L-2] Revert transaction when address.calls fails

There are 2 instances of this 
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L52C13-L56C20
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L65C13-L68C14

To prevent potential issues, it is important to revert early as soon as an `address.call` fails in a function. The `execute(...)` function of `UTBExecutor.sol ` does not revert the transaction when an `address.call` fails.

```
File: UTBExecutor.sol
41: function execute(
        ...//Some lines omitted for clarity
49:    ) public onlyOwner {
...//Some lines omitted for clarity

51: if (token == address(0)) {
52:            (success, ) = target.call{value: amount}(payload);
53:            if (!success) {
54:                (refund.call{value: amount}(""));//@audit revert early
55:            }
56:            return;
57:        }
58:
59:        uint initBalance = IERC20(token).balanceOf(address(this));
60:
61:        IERC20(token).transferFrom(msg.sender, address(this), amount);
62:        IERC20(token).approve(paymentOperator, amount);
63:
64:        if (extraNative > 0) {
65:            (success, ) = target.call{value: extraNative}(payload);
66:            if (!success) {
67:                (refund.call{value: extraNative}(""));//@audit revert early.
68:            }
69:        } else {
70:            (success, ) = target.call(payload);//@audit no success check
71:        }
...
```
It is important to revert the transaction early when there is a failed `address.call`

Impact: 
If for example the transaction in line 65 above fails, the transaction only  sends back the `extraNative` ETH and continues execution. There is no way to tell what succeeds and what does not until the sender manually compares there balance. This opens up to assumptions which can lead to potential issues.


Recommendation: 
Implement a revert whenever the boolean return value from an `address.call` reverts in a function

```
(success, ) = target.call{value: extraNative}(payload);
            if (!success) revert("call failed");
```


## [L-3] Prevent gas griefing attacks that's possible with custom `address.call`

There are 3 instances of this
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L52
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L65
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L70

Whenever the returned `bytes data` is not required, using the `.call()` function with non TRUSTED addresses opens the transaction to unnecessary gas griefing by return huge `bytes data`.

Note that this 
```
(success, ) = target.call{value: amount}(payload);
```
is the same thing as this
```
(success, bytes data ) = target.call{value: amount}(payload);
```
So in both cases, the `bytes data` is returned and copied to memory. Malicious `target address` can return huge `bytes data` to cause gas grief to the sender. 

Impact: 
Malicious `target address` can gas grief the sender making the sender waste more gas than necessary.

Recommendation:
Short term: When returned data is not required, use a low level call.
```
bool status;
assembly {
    status := call(gas(), receiver, amount, 0, 0, 0, 0)
}
require(status, "call failed");
```
Long Term: Consider using https://github.com/nomad-xyz/ExcessivelySafeCall



## [L-4] Lack of address existence check
There are 3 instances of this
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L52
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L65
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L70

The `target` address in the `execute(...)` function was not checked for address existence before making a `.call(...)` to it.

According to Solidity docs: 

> The low-level functions call, delegatecall and staticcall return true as their >first return value if the account called is non-existent, as part of the design of >the EVM. Account existence must be checked prior to calling if needed.

Source: https://docs.soliditylang.org/en/latest/control-structures.html#error-handling-assert-require-revert-and-exceptions

Impact: if `target` address does not exist, the boolean return value from `.call(...)` will return true when infact the asset was not transferred.

Recommendation: 
Implement contract existence check before `address.call()`.
```
function doesContractExist(address contractAddress) external view returns (bool) {
        // Check if the contract's code size is greater than zero
        uint256 codeSize;
        assembly {
            codeSize := extcodesize(contractAddress)
        }
        return codeSize > 0;
    }
```

## [L-5] Contract `.call()` with the same `payload` is made to `target` address 3 times in the same function.

There is 1 instance of this
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBExecutor.sol#L51C9-L71C10

The call() function is made to the `target` address 3 different times with the same payload in the `UTBExecutor.sol#execute(...)` function.

Making 3 contract calls to the same address with the same payload is not efficient and can be refactored in to just 1 contract call since the payload is the same.

```
File: UTBExecutor.sol
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
@>            (success, ) = target.call{value: amount}(payload);//@audit 3 different calls with same payload.
            if (!success) {
                (refund.call{value: amount}(""));
            }
            return;
        }

        uint initBalance = IERC20(token).balanceOf(address(this));

        IERC20(token).transferFrom(msg.sender, address(this), amount);
        IERC20(token).approve(paymentOperator, amount);

        if (extraNative > 0) {
@>            (success, ) = target.call{value: extraNative}(payload);
            if (!success) {
                (refund.call{value: extraNative}(""));
            }
        } else {
@>            (success, ) = target.call(payload);
        }

        uint remainingBalance = IERC20(token).balanceOf(address(this)) -
            initBalance;

        if (remainingBalance == 0) {
            return;
        }

        IERC20(token).transfer(refund, remainingBalance);
    }

```

Impact: 
It wastes gas and reduces code readability and simplicity.

Recommendation:
Refactor the `target.call(payload)` so that the `target` contract call is done just once instead of 3 times.

```
if (token == address(0)) {
            (success, ) = target.call{value: amount + extraNative}(payload);
            require(success, "call failed");
        }
```
The above can replace the 3 `target.call(payload)` call since its the same `target` address and `payload`.


## [L-6] No `deadline` and address zero check for signatures

There is one instance of this
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBFeeCollector.sol#L44C5-L63C1

The `collectFees(...)` function of the UTBFeeCollector.sol contract verifies signature without a `deadline`.

Secondly there is no check to ensure the `recovered` address is not zero address. This is because ecrecover can return address zero for invalid signature.

Lastly there is no use of nonce for the signatures.
```
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

Impact: 
- Invalid signature can be used before the `signer` state variable is set since it will initially be zero. 
- Signature have no deadline and can be executed anytime

Recommendation:
Implement `deadline` to signatures and also add address zero check on the recovered address. 
```
require(blocktimestamp >= deadline, "signature expired");
require(recovered != address(0), "signature expired");
```
See EIP712: https://eips.ethereum.org/EIPS/eip-712


## [L-7] `ecrecover` is susceptible to signature malleability
There is 1 instance of this
- https://github.com/code-423n4/2024-01-decent/blob/07ef78215e3d246d47a410651906287c6acec3ef/src/UTBFeeCollector.sol#L53

The `collectFees(...)` function uses built in `ecrecover(...)` to recover the address of the signer. The `ecrecover(...)` function is succeptible to signature malleability

```
File: UTBFeeCollector.sol
function collectFees(
        FeeStructure calldata fees,
        bytes memory packedInfo,
        bytes memory signature
    ) public payable onlyUtb {
        bytes32 constructedHash = keccak256(
            abi.encodePacked(BANNER, keccak256(packedInfo))
        );
        (bytes32 r, bytes32 s, uint8 v) = splitSignature(signature);//@audit no deadline for signature
        address recovered = ecrecover(constructedHash, v, r, s);
        require(recovered == signer, "Wrong signature");//@audit  check that sig is not zero addresss.
        if (fees.feeToken != address(0)) {
            IERC20(fees.feeToken).transferFrom(
                utb,
                address(this),
                fees.feeAmount
            );
        }
    }
```
Impact: 

Signature malleability leading to potential signature replay attack since there is even no nonce implemented.

Recommendation:
Consider using openzeppelin's `ECDSA.ecrecover(...)` function instead of the built in `ecrecover`.

OZ's ECDSA: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/e5c63635e3508a8d9d0afed091578cc4bb59a9c7/contracts/utils/cryptography/ECDSA.sol#L154

