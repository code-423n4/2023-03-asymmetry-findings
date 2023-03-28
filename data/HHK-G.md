### Custom errors instead of requires

```require(pauseStaking == false, "staking is paused");```
```if(pauseStaking == false) revert StakingPaused();```

### In SafETH Stake() can have everything in the same loop.

### Line 88 of SafEth.sol, ```if (weight == 0) continue``` should be one line above

This would save a sload as we wouldn't need to load ```IDerivative derivative = derivatives[i];``` when weight == 0

### All loops can move i++ in unckecked tags

### Using a state variable in *for* condition and reading state variable multiple time should be avoided

Ex: https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L84

Declaring a memory d = derivativeCount and then using it in the loop would save gas as each loop will mload instead of sload. 
Also because derivativeCount is used 2 times in this function it would save an extra sload to declare d at the beginning.

### rETH ethPerDerivative() does a useless multiplication and division

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L215

Multiply by 1e18 and then instantly divide by 1e18 is a waste of gas.

### In SafETH addDerivative() ```localTotalWeight``` should be calculated at the beginning

Calculate ```localTotalWeight``` first and then add ```_weight``` to get the new totalWeight, this will save one loop and one sload.

### In SafETH rebalanceToWeights() the check ```ethAmountToRebalance == 0``` is useless

It should be checked before the loop and return or not checked at all since there is low chance this function gets called if there is no eth to rebalance.