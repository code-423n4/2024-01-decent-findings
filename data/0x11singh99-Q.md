# Quality Assurance Report

| QA Issues | Issues                                         | Instances |
| --------- | ---------------------------------------------- | --------- |
| [[L-01](#l-01-check-for-address0-before-assigning-in-state-variable)] | Check for address(0) before assigning in state variable | 8 |
| [[L-02](#l-02-missing-address0-check-in-constructor)] | Missing `address(0)` check in `constructor` | 3 |
| [[N-01](#n-01-if-any-function-is-not-being-called-inside-the-contract-make-it-external)] | If any function is not being called inside the contract, make it external. | 8 |
| [[N-02](#n-02-unnecessary-type-casting)] | Unnecessary Type casting | 1 |

# Low

## [L-01] Check for address(0) before assigning in state variable

**Note: Missed by bot-report**

_8 instances_

<details><summary>see instance</summary>

```solidity
File : src/DcntEth.sol

20:    function setRouter(address _router) public {
21:          router = _router; //@audit check `_router` for address(0)
22:    }

```
[20-22](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L20C5-L22C6)

```solidity
File : src/DecentEthRouter.sol

68:     function registerDcntEth(address _addr) public onlyOwner {
69:          dcntEth = IDcntEth(_addr); //@audit check `_addr` for address(0)
70:     }

```
[68-70](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L68C4-L70C6)

```solidity
File : src/UTB.sol

28:     function setExecutor(address _executor) public onlyOwner {
29:          executor = IUTBExecutor(_executor); //@audit check `_executor` for address(0)
30:     }


36:     function setWrapped(address payable _wrapped) public onlyOwner {
37:          wrapped = IWETH(_wrapped); //@audit check `_wrapped` for address(0)
38:     }


44:     function setFeeCollector(address payable _feeCollector) public onlyOwner {
45:          feeCollector = IUTBFeeCollector(_feeCollector); //@audit check `_feeCollector` for address(0)
46:     }


325:    function registerSwapper(address swapper) public onlyOwner {
326:         ISwapper s = ISwapper(swapper);
327:         swappers[s.getId()] = swapper; //@audit check `swapper` for address(0)
328:     }


334:    function registerBridge(address bridge) public onlyOwner {
335:         IBridgeAdapter b = IBridgeAdapter(bridge);
336:         bridgeAdapters[b.getId()] = bridge; //@audit check `bridge` for address(0)
337:     }

```
[28-30](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L28C5-L30C6), [36-38](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L36C1-L38C6), [44-46](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L44C5-L46C6), [325-328](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L325C5-L328C6), [334-337](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol#L334C4-L337C6)

```solidity
File : src/UTBFeeCollector.sol

18:     function setSigner(address _signer) public onlyOwner {
19:          signer = _signer; //@audit check `_signer` for address(0)
20:     }

```
[18-20](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L18C5-L20C6)
</details>

## [L-02] Missing `address(0)` check in `constructor`

In a smart contract, an address might be empty or "zero" at the start. If you don't check whether an address is empty before using it, the contract might get confused or make mistakes.

_These instances are present in the bot report but instead of checking the address, the bool variable has been checked._

**Note: Missed by bot-report**

_3 instances_

<details><summary>see instance</summary>

```solidity
File : src/DecentBridgeExecutor.sol

12:    constructor(address _weth, bool gasIsEth) Owned(msg.sender) {
13:         weth = IWETH(payable(_weth)); //@audit check for `address(0)`
14:         gasCurrencyIsEth = gasIsEth;
15:     }

```
[12-15](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol#L12C4-L15C6)

```solidity
File : src/DecentEthRouter.sol

27:     constructor(
28:         address payable _wethAddress,
29:         bool gasIsEth,
30:         address _executor
31:     ) Owned(msg.sender) {
32:         weth = IWETH(_wethAddress); //@audit check for `address(0)`
33:         gasCurrencyIsEth = gasIsEth;
34:         executor = IDecentBridgeExecutor(payable(_executor)); //@audit check for `address(0)`
35:     }

```
[27-35](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol#L27C4-L35C6)
</details>

# Non-Critical

## [N-01] If any function is not being called inside the contract, make it external.

**Note: Missed by bot-report**

_8 instances_

<details><summary>see instance</summary>

```solidity
File : src/DcntEth.sol

20:    function setRouter(address _router) public {

24:    function mint(address _to, uint256 _amount) public onlyRouter {    

28:    function burn(address _from, uint256 _amount) public onlyRouter {

32:    function mintByOwner(address _to, uint256 _amount) public onlyOwner {

36:    function burnByOwner(address _from, uint256 _amount) public onlyOwner {

```
[20](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L20C4-L20C49), [24](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L24C5-L24C68), [28](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L28C5-L28C70), [32](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L32C5-L32C74), [36](https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L36C5-L36C76)

```solidity
File : src/UTBFeeCollector.sol

18:    function setSigner(address _signer) public onlyOwner {

44:    function collectFees(
45:        FeeStructure calldata fees,
46:        bytes memory packedInfo,
47:        bytes memory signature
48:    ) public payable onlyUtb {

69:    function redeemFees(address token, uint amount) public onlyOwner {

```
[18](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L18C5-L18C59), [44-48](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L44C1-L48C31), [69](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol#L69C5-L69C71)
</details>

## [N-02] Unnecessary Type casting

_1 instance_

```solidity
File : src/bridge_adapters/BaseAdapter.sol

12:         require(
13:             msg.sender == address(bridgeExecutor), //@audit `bridgeExecutor` is already type of address
14:             "Only bridge executor can call this"
15:         );

```
[12-15](https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/BaseAdapter.sol#L12C1-L15C11)