[G-1] Don’t compare boolean expressions to boolean literals
====================
`if (<x> == true) => if (<x>), if (<x> == false) => if (!<x>).`

```solidity
File: contracts/SafEth/SafEth.sol
64:         require(pauseStaking == false, "staking is paused");
```

```solidity
File: contracts/SafEth/SafEth.sol
109:         require(pauseUnstaking == false, "unstaking is paused");
```

[G-2] `++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for` loops
===
[The increment in for loop post condition can be made unchecked](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)
```solidity
File: contracts/SafEth/SafEth.sol
71:         for (uint i = 0; i < derivativeCount; i++)
```
```solidity
File: contracts/SafEth/SafEth.sol
84:         for (uint i = 0; i < derivativeCount; i++) {
```
```solidity
File: contracts/SafEth/SafEth.sol
113:         for (uint256 i = 0; i < derivativeCount; i++) {
```
```solidity
File: contracts/SafEth/SafEth.sol
140:         for (uint i = 0; i < derivativeCount; i++) {
```
```solidity
File: contracts/SafEth/SafEth.sol
147:         for (uint i = 0; i < derivativeCount; i++) {
```
```solidity
File: contracts/SafEth/SafEth.sol
171:         for (uint256 i = 0; i < derivativeCount; i++)
```
```solidity
File: contracts/SafEth/SafEth.sol
191:         for (uint256 i = 0; i < derivativeCount; i++)
```
[G-3] Multiple accesses of a mapping should use a local variable cache
===
`derivatives` mapping
---
```solidity
File: contracts/SafEth/SafEth.sol
71:         for (uint i = 0; i < derivativeCount; i++)
72:             underlyingValue +=
73:                 (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74:                     derivatives[i].balance()) /
75:                 10 ** 18;
```
[G-4] State variables should be cached in local variables
===
`derivativeCount` variable in `stake()` function
---
```solidity
File: contracts/SafEth/SafEth.sol
71:         for (uint i = 0; i < derivativeCount; i++)
File: contracts/SafEth/SafEth.sol
84:         for (uint i = 0; i < derivativeCount; i++) {
```
`derivativeCount` variable in `rebalanceToWeights()` function
---
```solidity
File: contracts/SafEth/SafEth.sol
140:         for (uint i = 0; i < derivativeCount; i++) {
File: contracts/SafEth/SafEth.sol
147:         for (uint i = 0; i < derivativeCount; i++) {
```
`derivativeCount` variable in `addDerivative()` function
---
```solidity
File: contracts/SafEth/SafEth.sol
186:         derivatives[derivativeCount] = IDerivative(_contractAddress);
187:         weights[derivativeCount] = _weight;
188:         derivativeCount++;
189: 
190:         uint256 localTotalWeight = 0;
191:         for (uint256 i = 0; i < derivativeCount; i++)
192:             localTotalWeight += weights[i];
193:         totalWeight = localTotalWeight;
194:         emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
```