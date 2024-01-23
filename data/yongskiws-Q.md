# The constructor of the contract sets the owner to msg.sender

without any checks. If the contract is initialized by an insecure account, an attacker could potentially gain ownership of the contract.

``` solidity
decent-bridge\DecentEthRouter.sol
27:    constructor(
28:         address payable _wethAddress,
29:         bool gasIsEth,
30:         address _executor
31:     ) Owned(msg.sender) {
32:         weth = IWETH(_wethAddress);
33:         gasCurrencyIsEth = gasIsEth;
34:         executor = IDecentBridgeExecutor(payable(_executor));
35:     }
36: 

12:     constructor(address _weth, bool gasIsEth) Owned(msg.sender) {
13:         weth = IWETH(payable(_weth));
14:         gasCurrencyIsEth = gasIsEth;
15:     }

UTB.sol
16:     constructor() Owned(msg.sender) {}

UTBExecutor.sol
8:     constructor() Owned(msg.sender) {}


```

Recommendation:

``` solidity
constructor() Owned(msg.sender) {
    require(isAuthorizedAddress(msg.sender), "Unauthorized address");
}
```

# Consider add role management

The SET functions are protected by the onlyOwner modifier, which is a good practice. However, there is no role management and Implement a role management system to ensure that only authorized addresses. This could potentially lead to issues if the contract is initialized by an insecure account.

``` solidity
UTB.sol
28:     function setExecutor(address _executor) public onlyOwner {
29:         executor = IUTBExecutor(_executor);
30:     }


36:     function setWrapped(address payable _wrapped) public onlyOwner {
37:         wrapped = IWETH(_wrapped);
38:     }

44:    function setFeeCollector(address payable _feeCollector) public onlyOwner {
45:         feeCollector = IUTBFeeCollector(_feeCollector);
46:     }

UTBFeeCollector.sol
18:     function setSigner(address _signer) public onlyOwner {
19:         signer = _signer;
20:     }

bridge_adapters\BaseAdapter.sol
19:     function setBridgeExecutor(address _executor) public onlyOwner {
20:         bridgeExecutor = _executor;
21:     }

bridge_adapters\DecentBridgeAdapter.sol
26:     function setRouter(address _router) public onlyOwner {
27:         router = IDecentEthRouter(payable(_router));
28:     }

bridge_adapters\StargateBridgeAdapter.sol
33:     function setRouter(address _router) public onlyOwner {
34:         router = IStargateRouter(_router);
35:     }

bridge_adapters\StargateBridgeAdapter.sol
37:     function setStargateEth(address _sgEth) public onlyOwner {
38:         stargateEth = _sgEth;
39:     }

swappers\UniSwapper.sol
20:     function setRouter(address _router) public onlyOwner {
21:         uniswap_router = _router;
22:     }
23: 
24:     function setWrapped(address payable _wrapped) public onlyOwner {
25:         wrapped = _wrapped;
26:     }
```

# Consider use nonReentrant from OpenZeppelin

An attacker can recursively call this function before the balance of the contract is updated, leading to an infinite loop and draining the contract's funds.

Recommendation: Use the nonReentrant modifier from OpenZeppelin's library or implement a similar mechanism to prevent reentrancy.

``` solidity
function onOFTReceived(
    uint16 _srcChainId,
    bytes calldata,
    uint64,
    bytes32,
    uint _amount,
    bytes memory _payload
) external override onlyLzApp {
    // ...
    if (weth.balanceOf(address(this)) < _amount) {
        dcntEth.transfer(_to, _amount);
        return;
    }
    // ...
```

