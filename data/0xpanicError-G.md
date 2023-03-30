https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

# Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Intialise non-zero variables at the time of declaration | 1 |
| [GAS-2](#GAS-2) | Cache array length in memory when called in loops | 5 |
| [GAS-3](#GAS-3) | If value is read only once, load directly from storage | 2 |
| [GAS-4](#GAS-4) | No need to loop to add or adjust totalWeight | 2 |
### [GAS-1] Intialise non-zero variables at the time of declaration
If a variable will hold a non-zero value then it should be set to a non-zero value at time of declaration.

`preDepositPrice` is declared and later set to either `10 ** 18` or `(10 ** 18 * underlyingValue) / totalSupply`. Instead it can be set to `10**18` and
if total supply is not zero, set to `(10 ** 18 * underlyingValue) / totalSuppl`.

*SSTORE requires 20000 gas from zero to non-zero but only 2900 from non-zero to non-zero*

*Instances (1)*:
```solidity
File: contracts/SafEth/SafEth.sol

78:   uint256 preDepositPrice; // Price of safETH in regards to ETH

79:   if (totalSupply == 0)

80:      preDepositPrice = 10 ** 18; // initializes with a price of 1

81:   else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
```

### [GAS-2] Cache array length in memory when called in loops
Whenever looping through an array, cache the array length in memory to save gas.

In the given loops, `derivativesCount` is called from storage for each iteration. Instead set `derivativesCount` in memory and call that
in loops to save gas inside each function.

*SLOAD requires 100 gas while MLOAD only requires 3 gas*

*Instances (5)*:
```solidity
File: contracts/SafEth/SafEth.sol

71:   for (uint i = 0; i < derivativeCount; i++)

84:   for (uint i = 0; i < derivativeCount; i++)

113:  for (uint256 i = 0; i < derivativeCount; i++)

140:  for (uint i = 0; i < derivativeCount; i++)

147:  for (uint i = 0; i < derivativeCount; i++)
```

### [GAS-3] If value is read only once, load directly from storage
If a storage variable is only read once, it's cheaper to read directly rather than storing it in memory.

`totalSupply()` is called only once but it's value is stored in memory and then read in the if statement. Instead
the statement can directly read from storage `if(!totalSupply())` and not call MSTORE and MLOAD.

*Additional MSTORE and MLOAD take 6 gas*

*Instances (2)*:
```solidity
File: contracts/SafEth/SafEth.sol

77:   uint256 totalSupply = totalSupply();

79:   if (totalSupply == 0)

```
```solidity
File: contracts/SafEth/SafEth.sol

110:   uint256 safEthTotalSupply = totalSupply();

116:   _safEthAmount) / safEthTotalSupply;

```

### [GAS-4] No need to loop to add or adjust totalWeight

`totalWeight` can directly be changed whenever a weights are adjusted or added. In the `adjustWeight` function, `totalWeight` could be set to `totalWeight + _weight - weights[_derivativeIndex]` before setting the mapping
to the new value.

Similarly in `addDerivative` function, `totalWeight` could be set to `totalWeight + _weight`.

* Prevents unneccesary operations and saves gas

*Instances (2)*:
```solidity
File: contracts/SafEth/SafEth.sol

170:   uint256 localTotalWeight = 0;

171:   for (uint256 i = 0; i < derivativeCount; i++)

172:   localTotalWeight += weights[i];

173:   totalWeight = localTotalWeight;

```
```solidity
File: contracts/SafEth/SafEth.sol

190:   uint256 localTotalWeight = 0;

191:   for (uint256 i = 0; i < derivativeCount; i++)

192:   localTotalWeight += weights[i];

193:   totalWeight = localTotalWeight;
```