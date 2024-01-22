## G use assembly to set address

`file:src/UTB.sol`
```solidity
function setExecutor(address _executor) public onlyOwner {
        executor = IUTBExecutor(_executor);
    }


function setWrapped(address payable _wrapped) public onlyOwner {
        wrapped = IWETH(_wrapped);
    }

function setFeeCollector(address payable _feeCollector) public onlyOwner {
        feeCollector = IUTBFeeCollector(_feeCollector);
    }
```


`file:src/UTBFeeCollector.sol`
```solidity
function setSigner(address _signer) public onlyOwner {
        signer = _signer;
    }
```

`file:src/bridge_adapters/StargateBridgeAdapter.sol`

```solidity
function setRouter(address _router) public onlyOwner {
        router = IStargateRouter(_router);
    }

    function setStargateEth(address _sgEth) public onlyOwner {
        stargateEth = _sgEth;
    }
```

`file:src/swappers/UniSwapper.sol`
```solidity
function setRouter(address _router) public onlyOwner {
        uniswap_router = _router;
    }

    function setWrapped(address payable _wrapped) public onlyOwner {
        wrapped = _wrapped;
    }
```

`file:src/bridge_adapters/BaseAdapter.sol`
```solidity
function setBridgeExecutor(address _executor) public onlyOwner {
        bridgeExecutor = _exec`utor;
    }
```

## G use assembly to set mapping

`file:src/UTB.sol`
```solidity
function registerSwapper(address swapper) public onlyOwner {
        ISwapper s = ISwapper(swapper);
        swappers[s.getId()] = swapper;
    }

    /**
     * @dev Registers and maps a bridge adapter to a bridge adapter ID.
     * @param bridge The address of the bridge adapter.
     */
    
    function registerBridge(address bridge) public onlyOwner {
        IBridgeAdapter b = IBridgeAdapter(bridge);
        bridgeAdapters[b.getId()] = bridge;
    }
```

`file:src/bridge_adapters/DecentBridgeAdapter.sol`

```solidity
function registerRemoteBridgeAdapter(
        uint256 dstChainId,
        uint16 dstLzId,
        address decentBridgeAdapter
    ) public onlyOwner {
        lzIdLookup[dstChainId] = dstLzId;
        chainIdLookup[dstLzId] = dstChainId;
        destinationBridgeAdapter[dstChainId] = decentBridgeAdapter;
    }
```
`file:src/bridge_adapters/StargateBridgeAdapter.sol`
```solidity
function registerRemoteBridgeAdapter(
        uint256 dstChainId,
        uint16 dstLzId,
        address decentBridgeAdapter
    ) public onlyOwner {
        lzIdLookup[dstChainId] = dstLzId;
        chainIdLookup[dstLzId] = dstChainId;
        destinationBridgeAdapter[dstChainId] = decentBridgeAdapter;
    }
```
## G-3 use low level call

`file:src/UTBFeeCollector.sol`
```solidity
function redeemFees(address token, uint amount) public onlyOwner {
        if (token == address(0)) {
            payable(owner).transfer(amount);
        } else {
            IERC20(token).transfer(owner, amount);
        }
    }
```
## G-4 can be set to immutable 

`file:src/bridge_adapters/DecentBridgeAdapter.sol`

```solidity
bool gasIsEth;
address bridgeToken;
```

function getBridgedAmount can be removed

`file:src/bridge_adapters/DecentBridgeAdapter.sol`
```solidity
function getBridgedAmount(
        uint256 amt2Bridge,
        address /*tokenIn*/,
        address /*tokenOut*/
    ) external pure returns (uint256) {
        return amt2Bridge;
    }
```
## G-5 destinationBridgeAdapter[dstChainId] can be cached

`file:src/bridge_adapters/DecentBridgeAdapter.sol`
```solidity
function bridge(
        uint256 amt2Bridge,
        SwapInstructions memory postBridge,
        uint256 dstChainId,
        address target,
        address paymentOperator,
        bytes memory payload,
        bytes calldata additionalArgs,
        address payable refund
    ) public payable onlyUtb returns (bytes memory bridgePayload) {
        require(
            destinationBridgeAdapter[dstChainId] != address(0),
            string.concat("dst chain address not set ")
        );

        uint64 dstGas = abi.decode(additionalArgs, (uint64));

        bridgePayload = abi.encodeCall(
            this.receiveFromBridge,
            (postBridge, target, paymentOperator, payload, refund)
        );

        SwapParams memory swapParams = abi.decode(
            postBridge.swapPayload,
            (SwapParams)
        );

        if (!gasIsEth) {
            IERC20(bridgeToken).transferFrom(
                msg.sender,
                address(this),
                amt2Bridge
            );
            IERC20(bridgeToken).approve(address(router), amt2Bridge);
        }

        router.bridgeWithPayload{value: msg.value}(
            lzIdLookup[dstChainId],
            destinationBridgeAdapter[dstChainId],
            swapParams.amountIn,
            false,
            dstGas,
            bridgePayload
        );
    }
```