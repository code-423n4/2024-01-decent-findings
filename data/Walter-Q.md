1) ## missing check of given swapParams.amountOut
UniSwapper#134 wont check the potential value of this variable resulting in possible loss of funds due to lack of slippage
https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/swappers/UniSwapper.sol#L134
2) https://github.com/code-423n4/2024-01-decent/blob/011f62059f3a0b1f3577c8ccd1140f0cf3e7bb29/src/UTBFeeCollector.sol#L26-L36
there're no check on the validity of the given signature like the v or the result different from address(0)