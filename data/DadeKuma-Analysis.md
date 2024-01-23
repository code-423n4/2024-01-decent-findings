# Decent Analysis

## Summary

| Id  | Title                      |
|:---:|:---------------------------|
| [01](#01-high-level-architecture) | High-level architecture |
| [02](#02-analysis-of-the-codebase) | Analysis of the codebase |
| [03](#03-architecture-feedback) | Architecture feedback |
| [04](#04-centralization-risks) | Centralization risks |
| [05](#05-systemic-risks) | Systemic risks |

## [01] High-level architecture

### Bridge Sequence Diagram
![Bridge](https://i.imgur.com/x0XUc6K.png)

### Swap Sequence Diagram
![Swap](https://i.imgur.com/ELmat2m.png)

## [02] Analysis of the codebase

- `UTBExecutor` and `DecentBridgeExecutor` can perform arbitrary calls without any limits. It doesn't appear to be an immediate risk, but it's worth noting that this could pose a risk in the future.
- The codebase quality is **high** and well-documented, following the best industry practices.
- Test quality is **medium**. While there are many E2E tests, there are few unit tests, which would have been useful for this audit. It is also recommended to add some fuzzing tests to cover more corner cases.

## [03] Architecture feedback

- The architecture, in general, is simple and elegant. It makes use of LayerZero, and `DcntEth` is a simple extension of the generic `OFTV2` dApp.
- `Executors` make use of arbitrary calls. Some functions are protected by an `onlyExecutor` modifier. It is recommended to remove all the arbitrary calls accessed by users to avoid any sort of issue.

## [04] Centralization risks

There are several centralization risks:

- The owner of `DcntEth` has unlimited power to mint and burn tokens. It's worth noting that those tokens are redeemable with `WETH` (anyone can provide liquidity, and the owner can theoretically sweep these funds).
- The owner can override and remove chains that were previously supported by the system.

## [05] Systemic risks

There are several systemic risks:

- Several dependencies are needed to ensure that the protocol will work correctly. The entire bridging logic is built on top of LayerZero, so it's essential to conduct extensive tests, especially for non-EVM chains, to ensure that everything works correctly.
- The system is using a library (`decent-bridge`) that may contain some issues, which is already deployed. It is recommended to fix any other part of the system already deployed that is using these libraries to avoid issues.
- Bridges are always a delicate topic. It's highly recommended to test every part of the system on a Testnet before doing the actual deployment.


### Time spent:
18 hours