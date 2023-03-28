## Gas
- GAS-01 - In `SafEth.sol:stake()`, the `if (weight == 0) continue;` check can be moved one line up to avoid an unnecessary SLOAD in the `true` case
