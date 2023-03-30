# Use `unchecked{}` for subtraction operations where operands cannot underflow because of a previous `require` statement
Using `unchecked{}` for operations that cannot underflow because of a previous `require` statement saves gas as it ignores the overflow/underflow checks.

*There is 1 instance of this issue:*
```solidity
File: Reth.sol
200:             require(rethBalance2 > rethBalance1, "No rETH was minted");
201:             uint256 rethMinted = rethBalance2 - rethBalance1; // @audit use unchecked{} cannot overflow because of previous require
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L200-L201