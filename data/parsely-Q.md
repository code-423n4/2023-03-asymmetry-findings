# [L-01] SWC-119 Shadowing State Variable
## Context
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L77

```solidity
uint256 totalSupply = totalSupply();
```
## Description
Shadowing of State variables breaks solidity coding standards according to SWC-119
https://swcregistry.io/docs/SWC-119

## Resolution
Change the variable name to align with coding standards