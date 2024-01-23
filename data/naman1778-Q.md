## [N-01] Lack of address(0) checks in the constructor

Zero-address check should be used in the constructors, to avoid the risk of setting smth as address(0) at deploying time.

There are 3 instances of this issue in 3 files:

    File: src/bridge_adapters/DecentBridgeAdapter.sol	

    21: constructor(bool _gasIsEth, address _bridgeToken) BaseAdapter() {

https://github.com/code-423n4/2024-01-decent/blob/main/src/bridge_adapters/DecentBridgeAdapter.sol

    File: lib/decent-bridge/src/DecentEthRouter.sol	

    27: constructor(
    28:     address payable _wethAddress,
    29:     bool gasIsEth,
    30:     address _executor
    31: ) Owned(msg.sender) {

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentEthRouter.sol

    File: lib/decent-bridge/src/DecentBridgeExecutor.sol	

    12: constructor(address _weth, bool gasIsEth) Owned(msg.sender) {

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DecentBridgeExecutor.sol

## [N-02] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

There is 1 instance of this issue in 1 file:

    File: src/UTBFeeCollector.sol	

    31: assembly {

https://github.com/code-423n4/2024-01-decent/blob/main/src/UTBFeeCollector.sol
