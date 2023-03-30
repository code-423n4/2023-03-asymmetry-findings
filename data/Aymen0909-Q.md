# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | Same derivative can be added multiple times | Low | 1 |
| 2 | `maxSlippage` should have a maximum upper bound | Low | 1 |
| 3 | `addDerivative` should contain non zero address check | Low | 1 |
| 4 | Avoid floating pragma where possible | NC |  |
| 5 | Use scientific notation | NC | 17 |

## Findings

### 1- Same derivative can be added multiple times :

#### Risk : Low

The `addDerivative` function does not check if the new derivative exists or not so the same derivative can be added multiple times (by accident), the impact of this error is that this derivative will have a bigger weight than it was intended to because the `derivativeCount` is incremented when adding a duplicate derivative thus when looping over all the derivatives for calculating the total weight the weight of this derivative is counted multiple times.

It's important to note that there is no way to remove duplicate derivative because if its weight is set to 0 this will also remove the original derivative (as his weight will also be set to 0).

As this issue can potential cause problems for the protocol working it should be addressed by adding a way to chack for the existance of derivative contracts. 

#### Mitigation
Add a checks in the `addDerivative` function to verify the existance of the derivative.


### 2- `maxSlippage` should have a maximum upper bound  :

#### Risk : Low

The value of `maxSlippage` doesn't have a known constant maximum value so the owner can set it to any value between 0 and 1e18 (can't be higher due underflow errors), so when a user stakes his funds he doesn't really know what would be the max slippage when he intend to withdraw for example when a user stakes the max slippage was set to 1% which the user accept but at the moment he wants to unstake the max slippage value is now 8% which is too high for him.

To avoid any confusion the `maxSlippage` should have a constant upper bound value set in each derivative contract, so the users always know in advance the maximum slippage value they are risking.

#### Mitigation
You should considere adding a constant upper bound value for `maxSlippage` in each derivative contract.


### 3- `addDerivative` should contain non zero address check :

#### Risk : Low

The `addDerivative` function is used to add new derivative contract to the staking process, this function does not contain a check on the input derivative contract address `_contractAddress` which means it can be set to `address(0)` by accident and this will still sets the derivative weight and increment the `derivativeCount` and `totalWeight`, this can have a big negative impact on the protocol as when users will stakes their funds part of them will go to `address(0)` (get burned) and can't be withdrawn afterwards leading to a loss of funds.

The risk of this issue happening is very low but due to its bad impact it should be addressed.

#### Mitigation
Add non-zero address checks in the `addDerivative` function for the value of `_contractAddress`.


### 4- Avoid floating pragma where possible :

#### Risk : Non critical

All the contracts of the protocol use a floating solidity version `pragma solidity ^0.8.13` which should be fixed to agiven version (preferably to a recent one) to avoid any issues in the future, as locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

Note that pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or a package.


### 5- Use scientific notation :

When using multiples of 10 you shouldn't use decimal literals or exponentiation (e.g. 1000000, 10**18) but use scientific notation for better readability.

#### Risk : NON CRITICAL

#### Proof of Concept

Instances include:

10 ** 16 :

Reth.sol : [Line 44](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L44)

SfrxEth.sol : [Line 38](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L38)

WstEth.sol : [Line 35](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L35)

10 ** 17 :

SafEth.sol : [Line 54](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L54)

10 ** 18 : 

SafEth.sol : [Line 55](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L55), [Line 75](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L75), [Line 80](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L80), [Line 81](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L81), [Line 94](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L94), [Line 98](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L98) 

Reth.sol : [Line 173-174](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173-L174), [Line 214-215](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L214-L215)

SfrxEth.sol : [Line 74-75](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75), [Line 113-115](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L113-L115)

WstEth.sol : [Line 60](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L60), [Line 87](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L87)

10 ** 36 : 

Reth.sol : [Line 171](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L171)

#### Mitigation
Use scientific notation for the instances aforementioned.

