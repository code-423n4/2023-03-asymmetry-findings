# Report by MiksuJak

## Gas Optimizations

| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Store balance calls return values in `memory` instead of calling them multiple times in one function | 2 |
| [GAS-2](#GAS-2) | Store derivative address in `memory` whenever it is read from storage more than once | 3 |
| [GAS-3](#GAS-3) | Do not recalculate `localTotalWeights` when a `weight` is adjusted or added | 2 |

### [GAS-1] Store balance calls return values in `memory` instead of calling them multiple times in one function

Store `<address>.balance()` in a `memory` variable to save gas on addresses balance calls.

#### Instance 1:
__File: contracts/SafEth/SafEth.sol   | stake |   Lines: 71-75__

```solidity
for (uint i = 0; i < derivativeCount; i++)
	underlyingValue +=
		(derivatives[i].ethPerDerivative(derivatives[i].balance()) *
			derivatives[i].balance()) /
		10 ** 18;
```
Could be changed to:
```solidity
for (uint i = 0; i < derivativeCount; i++) {
	uint256 derivativeBalance = derivatives[i].balance();
	underlyingValue +=
		(derivatives[i].ethPerDerivative(derivativeBalance) *
			derivativeBalance) /
		10 ** 18;
}
```
Saves _8145 gas_ per call for __SafEth-Integration.test.ts:78__ test.

#### Instance 2:
__File: contracts/SafEth/SafEth.sol   | rebalanceToWeights |   Lines: 140-143__

```solidity
for (uint i = 0; i < derivativeCount; i++) {
	if (derivatives[i].balance() > 0)
		derivatives[i].withdraw(derivatives[i].balance());
}
```
Could be changed to:
```solidity
for (uint i = 0; i < derivativeCount; i++) {
	uint256 derivativeBalance = derivatives[i].balance();
	if (derivativeBalance > 0)
		derivatives[i].withdraw(derivativeBalance);
}
```
Saves _8154 gas_ for __SafEth-Integration.test.ts:94__ test.
Saves _4760 gas_ for __SafEth-Integration.test.ts:123__ test.

### [GAS-2] Store derivative address in `memory` whenever it is read from storage more than once

There are `for` loops iterating over `derivatives`  and performing multiple calls on each `derivative` in these loops. Store `derivative[i]` in `memory` whenever it is used more than once in a single iteration.

#### Instance 1:
__File: contracts/SafEth/SafEth.sol  | stake |   Lines: 71-75__

 ```solidity
for (uint i = 0; i < derivativeCount; i++)
	underlyingValue +=
		(derivatives[i].ethPerDerivative(derivatives[i].balance()) *
			derivatives[i].balance()) /
		10 ** 18;
```
Could be changed to:
```solidity
for (uint i = 0; i < derivativeCount; i++) {
	IDerivative derivative = derivatives[i];
	underlyingValue +=
		(derivative.ethPerDerivative(derivative.balance()) *
			derivative.balance()) /
		10 ** 18;
}
```
Saves _~537 gas_ per call for __SafEth-Integration.test.ts:78__ test.

#### Instance 2:
__File: contracts/SafEth/SafEth.sol   | unstake |   Lines: 115-118__

 ```solidity
for (uint256 i = 0; i < derivativeCount; i++) {
	IDerivative derivative = derivatives[i];
	// withdraw a percentage of each asset based on the amount of safETH
	uint256 derivativeAmount = (derivative.balance() *
		_safEthAmount) / safEthTotalSupply;
	if (derivativeAmount == 0) continue; // if derivative empty ignore
	derivative.withdraw(derivativeAmount);
}
```
Could be changed to:
```solidity
for (uint256 i = 0; i < derivativeCount; i++) {
	// withdraw a percentage of each asset based on the amount of safETH
	uint256 derivativeAmount = (derivatives[i].balance() *
		_safEthAmount) / safEthTotalSupply;
	if (derivativeAmount == 0) continue; // if derivative empty ignore
	derivatives[i].withdraw(derivativeAmount);
}
```
Saves _~501 gas_ per call for __SafEth-Integration.test.ts:86__ test.

#### Instance 3:
__File: contracts/SafEth/SafEth.sol   | rebalanceToWeights |   Lines: 140-143__

 ```solidity
for (uint i = 0; i < derivativeCount; i++) {
	if (derivatives[i].balance() > 0)
		derivatives[i].withdraw(derivatives[i].balance());
}
```
Could be changed to:
```solidity
for (uint i = 0; i < derivativeCount; i++) {
	if (derivatives[i].balance() > 0)
		derivatives[i].withdraw(derivatives[i].balance());
}
```
Saves _546 gas_ for __SafEth-Integration.test.ts:94__ test.
Saves _353 gas_ for __SafEth-Integration.test.ts:123__ test.


### [GAS-3] Do not recalculate `totalWeights` when a `weight` is adjusted or added

When a `weight` is adjusted or a new one is added, `localTotalWeight` are calculated iterating over `weights` array. It is not needed since we know both the value of `totalWeight` and a `weight` to be adjusted/added. Moreover `weights` array does not ever change without `totalWeight` being updated, so we know that `weights.sum()` is always equal to `totalWeight`.

#### Instance 1:
__File: contracts/SafEth/SafEth.sol   | adjustWeight |   Lines: 169-173__

```solidity
weights[_derivativeIndex] = _weight;
uint256 localTotalWeight = 0;
for (uint256 i = 0; i < derivativeCount; i++)
	localTotalWeight += weights[i];
totalWeight = localTotalWeight;
```
Could be changed to:
```solidity
totalWeight = totalWeight + _weight - weights[_derivativeIndex];
weights[_derivativeIndex] = _weight;
```
Saves _7284 gas_ per call.

#### Instance 2:
__File: contracts/SafEth/SafEth.sol   | addDerivative |   Lines: 169-173__

```solidity
weights[derivativeCount] = _weight;
derivativeCount++;

uint256 localTotalWeight = 0;
for (uint256 i = 0; i < derivativeCount; i++)
	localTotalWeight += weights[i];
totalWeight = localTotalWeight;
```
Could be changed to:
```solidity
weights[derivativeCount] = _weight;
derivativeCount++;
totalWeight += _weight;
```
Saves _506 gas_ + _2455 gas * `weights.length`_ per call.

