Custom errors are cheaper than revert strings (cheaper deployment cost and runtime cost when the revert condition is met)
```Solidity
src/bridge_adapters/BaseAdapter.sol:12:         require(msg.sender == address(bridgeExecutor),"Only bridge executor can call this");
```
I suggest replacing revert string with custom error.