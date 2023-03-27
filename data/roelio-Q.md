Slippage should probably be limited

- Problem: Both https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L52 and https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L49 https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L59 should probably be restricted to not allow extreme slippage which could be used to do a malicious arbitrage. 
- Solution: Have an upper bound to the slippage.
