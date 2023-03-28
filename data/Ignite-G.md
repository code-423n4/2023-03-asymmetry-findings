# Code4rena - Assymetic Gas

## Gas Optimizations
|  | Issue | Instances |
|---|---|---|
| Gas-1 | Emitting events with variables from memory rather than state variables can help reduce gas costs | 4 |
| Gas-2 | Declare local variables as part of the function instead of inside the function  | 1 |

### [Gas-1] Emitting events with variables from memory rather than state variables can help reduce gas costs
Using memory variable instead of state variable in the emit statement may be slightly more gas-efficient because it does not require the Solidity compiler to generate an additional `SLOAD` instruction to read the state variable from storage.

Instances(4):


```solidity
        emit ChangeMinAmount(minAmount);
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L216

```solidity
        emit ChangeMaxAmount(maxAmount);
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L225

```solidity
        emit StakingPaused(pauseStaking);
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L234

```solidity
        emit UnstakingPaused(pauseUnstaking);
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L243


### [Gas-2] Declare local variables as part of the function instead of inside the function
Declare local variables as part of the function instead of inside the function to save gas.

Instances(1):

```
    function deposit() external payable onlyOwner returns (uint256) {
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L73

