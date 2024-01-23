

# Gas Optimizations

| Number | Issue | Instencs |
|--------|-------|----------|
|[G-01]| Use uint256(1)/uint256(2) instead for true and false boolean states  | 1 |
|[G-02]| Don’t make variables public unless necessary  | 9 |
|[G-03]| Using assembly to revert with an error message  | 6 |
|[G-04]| Use assembly to calculate hashes  | 2 |
|[G-05]| Use assembly to write address storage values  | 4 |
|[G-06]| Avoid Unnecessary Use of Storage  | 1 |
|[G-07]| Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4  | 2 |
|[G-08]| Create immutable variable to avoid an external call  | 7 |
|[G-09]| Using storage instead of memory for structs/arrays saves gas   | 8 |
|[G-10]| Use assembly to reuse memory space when making more than one external call  | 4 |
|[G-11]| Validate all parameters first before performing any other operations if there’s no side effects  | 2 |
|[G-12]| Use hardcode address instead address(this)  | 11 |
|[G-13]| It is sometimes cheaper to cache calldata  | 6 |


## [G-01] Use uint256(1)/uint256(2) instead for true and false boolean states

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. see source:

```solidity
file: blob/main/src/bridge_adapters/DecentBridgeAdapter.sol

18    bool gasIsEth;

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L18


## [G-02] Don’t make variables public unless necessary

Issue Description - Making variables public comes with some overhead costs that can be avoided in many cases. A public variable implicitly creates a public getter function of the same name, increasing the contract size.

Proposed Optimization - Only mark variables as public if their values truly need to be readable by external contracts/users. Otherwise, use private or internal visibility.

Estimated Gas Savings - The savings from avoiding unnecessary public variables are small per transaction, around 5-10 gas. However, this adds up over many transactions targeting a contract with public state variables that don’t need to be public.

Code Snippets:

```solidity
file: blob/main/src/swappers/UniSwapper.sol

17   address public uniswap_router;

18   address payable public wrapped;

16   uint8 public constant SWAPPER_ID = 0;

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L17

```solidity
file: blob/main/src/bridge_adapters/DecentBridgeAdapter.sol

13  uint8 public constant BRIDGE_ID = 0;

15  IDecentEthRouter public router;

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L13

```solidity
file: blob/main/src/bridge_adapters/StargateBridgeAdapter.sol

31   IStargateRouter public router;

22   uint8 public constant BRIDGE_ID = 1;

23   uint8 public constant SG_FEE_BPS = 6;

24   address public stargateEth;

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L31

## [G-03] Using assembly to revert with an error message

Issue Description - When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

Estimated Gas Savings - Here’s an example:

```solidity

/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;
    constructor() {
        owner = msg.sender;
    }
    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}
/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;
    constructor() {
        owner = msg.sender;
    }
    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}

```
From the example above, we can see that we get a gas saving of over 300 gas when reverting with the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

Code Snippets:

```solidity
file: blob/main/src/UTB.sol

75    require(msg.value >= swapParams.amountIn, "not enough native");

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L75

```solidity
file: blob/main/src/UTBFeeCollector.sol

29   require(signature.length == 65, "Invalid signature length");

54   require(recovered == signer, "Wrong signature");

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L29

```solidity
file: blob/main/src/bridge_adapters/DecentBridgeAdapter.sol

91   require(
            destinationBridgeAdapter[dstChainId] != address(0),
            string.concat("dst chain address not set ")
        );
    
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L91-L94


```solidity
file: blob/main/src/bridge_adapters/StargateBridgeAdapter.sol

157   require(
            dstAddr != address(0),
            string.concat("dst chain address not set ")
        );

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L157-L160


```solidity
file:  blob/main/src/swappers/UniSwapper.sol

96    require(uniswap_router != address(0), "router not set");

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L96

## [G-04] Use assembly to calculate hashes

Saves 5000 deployment gas per instance and 374 runtime gas per instance.  ### Unoptimized ```solidity function solidityHash(uint256 a, uint256 b) public view { 	//unoptimized 	keccak256(abi.encodePacked(a, b)); } ```  ### Optimized ```solidity function assemblyHash(uint256 a, uint256 b) public view { 	//optimized 	assembly { 		mstore(0x00, a) 		mstore(0x20, b) 		let hashedVal := keccak256(0x00, 0x40) 	} } ```

```solidity
file: blob/main/src/UTBFeeCollector.sol

49  bytes32 constructedHash = keccak256(

50   abi.encodePacked(BANNER, keccak256(packedInfo))

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L49

## [G-05] Use assembly to write address storage values


```solidity
file: blob/main/src/UTBFeeCollector.sol

19   signer = _signer;

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L19


```solidity
file: blob/main/src/bridge_adapters/DecentBridgeAdapter.sol

23   bridgeToken = _bridgeToken;

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L23

```solidity
file: blob/main/src/bridge_adapters/StargateBridgeAdapter.sol

38   stargateEth = _sgEth;

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L38

```solidity
file: blob/main/src/swappers/UniSwapper.sol

25   wrapped = _wrapped;

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L25

## [G-06] Avoid Unnecessary Use of Storage

Writing data to storage is one of the most expensive operations in Solidity. Reduce gas costs by avoiding unnecessary writes. For example, move constant values to memory:

```solidity

// Expensive
string public constant NAME = "MyContract"; 

// Cheaper
string memory NAME = "MyContract";

```

Also be careful about storing large pieces of data on-chain. Use alternative patterns like IPFS for larger data.

Storing data costs 20,000 gas versus 3 gas for memory.
Minimize storage by using memory for fixed constants and temporary values.
Store large data like files off-chain (IPFS) and put hash on-chain.

```solidity
file: blob/main/src/UTBFeeCollector.sol

10   string constant BANNER = "\x19Ethereum Signed Message:\n32";

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L10


## [G-07] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

Issue Description - The abi.encodePacked() function is used to concatenate multiple arguments into a byte array prior to 0.8.4. However, since 0.8.4 the bytes.concat() function was introduced which performs the same role but is preferred since it avoids ABI encoding overhead.

Proposed Optimization - Replace uses of abi.encodePacked() with bytes.concat() where multiple arguments need to be concatenated into a byte array.

Estimated Gas Savings - Using bytes.concat() instead of abi.encodePacked() saves approximately 300-500 gas per concatenation by avoiding ABI encoding.

Code Snippet:

```solidity
file: blob/main/src/UTBFeeCollector.sol

50  abi.encodePacked(BANNER, keccak256(packedInfo))

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L50


```solidity
file:  blob/main/src/bridge_adapters/StargateBridgeAdapter.sol

178    abi.encodePacked(getDestAdapter(dstChainId)),

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L178

## [G-08] Create immutable variable to avoid an external call

Instead of performing an external call to get the root address each time _enableNode is invoked, we can perform this external call once in the constructor and store the root as an immutable variable. Doing this will save 1 external call each time _enableNode is invoked.


Code Snippets:

```solidity
file: blob/main/src/UTB.sol
 
67    ISwapper swapper = ISwapper(swappers[swapInstructions.swapperId]);

69    SwapParams memory swapParams = abi.decode(

181   SwapParams memory newPostSwapParams = abi.decode(

211   IBridgeAdapter bridgeAdapter = IBridgeAdapter(bridgeAdapters[instructions.bridgeId]);

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L67

```solidity
file: blob/main/src/bridge_adapters/DecentBridgeAdapter.sol

51    SwapParams memory swapParams = abi.decode(

103   SwapParams memory swapParams = abi.decode(

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L51

```solidity
file: blob/main/src/swappers/UniSwapper.sol

33   SwapParams memory newSwapParams,

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L33

## [G-09] Using storage instead of memory for structs/arrays saves gas 

```solidity
file: blob/main/src/swappers/UniSwapper.sol

129   IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
 
```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#129


```solidity 
file: blob/main/src/UTB.sol

69  SwapParams memory swapParams = abi.decode(

181  SwapParams memory newPostSwapParams = abi.decode(

271   BridgeInstructions memory updatedInstructions

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L69


```solidity
file: blob/main/src/bridge_adapters/DecentBridgeAdapter.sol

51   SwapParams memory swapParams = abi.decode(

103  SwapParams memory swapParams = abi.decode(

134  SwapParams memory swapParams = abi.decode(

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L51

```solidity
file: blob/main/src/bridge_adapters/StargateBridgeAdapter.sol

202   SwapParams memory swapParams = abi.decode(

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L202


## [G-10] Use assembly to reuse memory space when making more than one external call

Issue Description - When making external calls, the solidity compiler has to encode the function signature and arguments in memory. It does not clear or reuse memory, so it expands memory each time.

Proposed Optimization - Use inline assembly to reuse the same memory space for multiple external calls. Store the function selector and arguments without expanding memory further.

Estimated Gas Savings - Reusing memory can save thousands of gas compared to expanding on each call. The baseline memory expansion per call is 3,000 gas. With larger arguments or return data, the savings would be even greater.


```solidity

See the example below;
contract Called {
    function add(uint256 a, uint256 b) external pure returns(uint256) {
        return a + b;
    }
}
contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns(uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);
        uint256 res = res1 + res2;
        return res;
    }
}
contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns(uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }
            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)
            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)
            // add results
            let res := add(res1, res2)
            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}
```

We save approximately 2,000 gas by using the scratch space to store the function selector and its arguments and also reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00 before exiting the assembly block.

Code Snippets:


use assembly for  SwapParams memory swapParams


```solidity
file:  blob/main/src/bridge_adapters/DecentBridgeAdapter.sol

51         SwapParams memory swapParams = abi.decode(
      postBridge.swapPayload,
      (SwapParams)
        );
        return
            router.estimateSendAndCallFee(
                router.MT_ETH_TRANSFER_WITH_PAYLOAD(),
                lzIdLookup[dstChainId],
                target,
                swapParams.amountIn,
                dstGas,
                false,
                payload
            );
    }

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L51-L65

use assembly for  SwapParams memory swapParams

```solidity
file:  blob/main/src/UTB.sol
 
67          ISwapper swapper = ISwapper(swappers[swapInstructions.swapperId]);

        SwapParams memory swapParams = abi.decode(
            swapInstructions.swapPayload,
            (SwapParams)
        );

        if (swapParams.tokenIn == address(0)) {
            require(msg.value >= swapParams.amountIn, "not enough native");
            wrapped.deposit{value: swapParams.amountIn}();
            swapParams.tokenIn = address(wrapped);
            swapInstructions.swapPayload = swapper.updateSwapParams(
                swapParams,
                swapInstructions.swapPayload
            );
        } else if (retrieveTokenIn) {
            IERC20(swapParams.tokenIn).transferFrom(
                msg.sender,
                address(this),
                swapParams.amountIn
            );
        }

        IERC20(swapParams.tokenIn).approve(
            address(swapper),
            swapParams.amountIn
        );

        (tokenOut, amountOut) = swapper.swap(swapInstructions.swapPayload);

        if (tokenOut == address(0)) {
            wrapped.withdraw(amountOut);
        }
    }

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L67-L100

use assembly for  SwapParams memory newPostSwapParams

```solidity
file:  blob/main/src/UTB.sol

180     SwapParams memory newPostSwapParams = abi.decode(
            instructions.postBridge.swapPayload,
            (SwapParams)
        );

        newPostSwapParams.amountIn = IBridgeAdapter(
            bridgeAdapters[instructions.bridgeId]
        ).getBridgedAmount(amountOut, tokenOut, newPostSwapParams.tokenIn);

        updatedInstructions = instructions;

        updatedInstructions.postBridge.swapPayload = ISwapper(swappers[
            instructions.postBridge.swapperId
        ]).updateSwapParams(
            newPostSwapParams,
            instructions.postBridge.swapPayload
        );


```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L180-L197

use assembly for IBridgeAdapter bridgeAdapter

```solidity
file:    blob/main/src/UTB.sol

211          IBridgeAdapter bridgeAdapter = IBridgeAdapter(bridgeAdapters[instructions.bridgeId]);
        address bridgeToken = bridgeAdapter.getBridgeToken(
            instructions.additionalArgs
        );
        if (bridgeToken != address(0)) {
            IERC20(bridgeToken).approve(address(bridgeAdapter), amt2Bridge);
            return false;
        }
        return true;
    }

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L211-L220


## [G-11] Validate all parameters first before performing any other operations if there’s no side effects

```solidity
file: blob/main/src/UTB.sol

82   } else if (retrieveTokenIn) {
            IERC20(swapParams.tokenIn).transferFrom(
                msg.sender,
                address(this),
                swapParams.amountIn
            );
        }

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L82-L88


```solidity
file: blob/main/src/UTBFeeCollector.sol

55  if (fees.feeToken != address(0)) {
            IERC20(fees.feeToken).transferFrom(
                utb,
                address(this),
                fees.feeAmount
            );
        }

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L55-L61

## [G-12] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

References:

https://book.getfoundry.sh/reference/forge-std/compute-create-address https://twitter.com/transmissions11/status/1518507047943245824


```solidity
file: blob/main/src/bridge_adapters/StargateBridgeAdapter.sol

88   IERC20(bridgeToken).transferFrom(msg.sender, address(this), amt2Bridge);

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/StargateBridgeAdapter.sol#L88

```solidity
file: blob/main/src/UTB.sol

85   address(this),

238   address(this),

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L85


```solidity
file: blob/main/src/swappers/UniSwapper.sol

132   recipient: address(this),

152   recipient: address(this),

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/swappers/UniSwapper.sol#L132

```solidity
file: blob/main/src/UTBExecutor.sol

59   uint initBalance = IERC20(token).balanceOf(address(this));

61   IERC20(token).transferFrom(msg.sender, address(this), amount);

73   uint remainingBalance = IERC20(token).balanceOf(address(this))

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBExecutor.sol#L59

```solidity
file: blob/main/src/UTBFeeCollector.sol

58   address(this),

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L58

```solidity
file: blob/main/src/bridge_adapters/DecentBridgeAdapter.sol

111  address(this),

141  address(this),

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol#L111

## [G-13] It is sometimes cheaper to cache calldata

Issue Description - While the calldataload opcode is relatively cheap, directly using it in a loop or multiple times can still result in unnecessary bytecode. Caching the loaded calldata first may allow for optimization opportunities.

Proposed Optimization - Cache calldata values in a local variable after first load, then reference the local variable instead of repeatedly using calldataload.

Estimated Gas Savings - Exact savings vary depending on contract, but caching calldata parameters can save 5-20 gas per usage by avoiding extra calldataload opcodes. Larger functions with many parameter uses see more benefit.

Code Snippet:

```solidity
file: blob/main/src/UTB.sol

109   SwapAndExecuteInstructions calldata instructions,

110   FeeStructure calldata fees,

135   SwapInstructions memory swapInstructions,

229    FeeStructure calldata fees,

312   SwapInstructions memory postBridge,

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L109


```solidity
file: blob/main/src/UTBFeeCollector.sol

45   FeeStructure calldata fees,

```
https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L45