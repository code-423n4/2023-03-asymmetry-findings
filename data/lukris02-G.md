# Gas Optimizations Report for Asymmetry contest
## Overview
During the audit, 12 gas issues were found.  

№ | Title | Instance Count | Saved
--- | --- | --- | ---
G-1 | Cache state variables instead of reading them from storage multiple times | 16 | 1600
G-2 | Use a function parameter instead of a storage variable | 4 | 400
G-3 | Use local variable cache instead of accessing mapping or array multiple times | 7 | 700
G-4 | Break the if-statement Require | 1 | 400
G-5 | Unnecessary loop in ```adjustWeight()``` | 1 | 400
G-6 | Unnecessary loop in ```addDerivative()``` | 1 | 400
G-7 | Use unchecked blocks for subtractions where underflow is impossible | 1 | 35
G-8 | Implement a function for removing a derivative | 7 | 
G-9 | Do not call ```balance()``` functions twice | 2 | 
G-10 | Some expressions can be transformed | 2 | 
G-11 | Do not multiply and divide by the same number | 1 | 
G-12 | Use uint256(1) and uint256(2) for pausing | 2 | 

## Gas Optimizations Findings(12)
### G-1. Cache state variables instead of reading them from storage multiple times
##### Description
Memory read is much cheaper than storage read.
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71 *2 (due to loop, if we take derivativeCount = 3)
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84 *2 (due to loop, if we take derivativeCount = 3)
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113 *2 (due to loop, if we take derivativeCount = 3)
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140 *2 (due to loop, if we take derivativeCount = 3)
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147 *2 (due to loop, if we take derivativeCount = 3)
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171 *2 (due to loop, if we take derivativeCount = 3)
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191 *2 (due to loop, if we take derivativeCount = 3)
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L187 
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L194 

##### Saved
This saves ~100.  
So, ~100*16 = 1600
#
### G-2. Use a function parameter instead of a storage variable
##### Description
Memory read is much cheaper than storage read.
##### Instances
- [```emit ChangeMinAmount(minAmount);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L216) use _minAmount
- [```emit ChangeMaxAmount(maxAmount);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L225) use _maxAmount
- [```emit StakingPaused(pauseStaking);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L234) use _pause
- [```emit UnstakingPaused(pauseUnstaking);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L243) use _pause

##### Saved
This saves ~100.  
So, ~100*4 = 400
#
### G-3. Use local variable cache instead of accessing mapping or array multiple times
##### Description
It saves gas due to not having to perform the same key’s keccak256 hash and/or offset recalculation.
##### Instances
- [```(derivatives[i].ethPerDerivative(derivatives[i].balance()) *```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L73) (cache derivatives[i])
- [```derivatives[i].balance()) /```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L74) (cache derivatives[i])
- [```derivatives[i].withdraw(derivativeAmount);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L118) (cache derivatives[i])
- [```derivatives[i].withdraw(derivatives[i].balance());```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L142) (cache derivatives[i]) * 2 
- [```uint256 ethAmount = (ethAmountToRebalance * weights[i]) /```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L149) (cache weights[i])
- [```derivatives[i].deposit{value: ethAmount}();```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L152) (cache derivatives[i])

##### Saved
This saves ~100.  
So, ~100*7 = 700
#
### G-4. Break the if-statement
##### Description
Break this condition into 2 parts. ```if (ethAmountToRebalance == 0) break;``` to do not iterate the loop for no reason.
##### Instances
- [```if (weights[i] == 0 || ethAmountToRebalance == 0) continue;```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L148) 

##### Recommendation
Change to:
```
if (ethAmountToRebalance == 0) break;
for (uint i = 0; i < derivativeCount; i++) {
    if (weights[i] == 0) continue;
    ...
```

##### Saved
This saves ~400 (if we take derivativeCount = 3).
#
### G-5. Unnecessary loop in ```adjustWeight()```
##### Description
There is no need for the loop.
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L170-L173

##### Recommendation
Instead of loop, consider just calculating the difference between previous and new weight. Change to:
```
if (_weight > oldWeight) totalWeight += _weight;
else totalWeight -= _weight;
```

##### Saved
This saves ~400 (if we take derivativeCount = 3).
#
### G-6. Unnecessary loop in ```addDerivative()```
##### Description
There is no need for the loop.
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L190-L193

##### Recommendation
Instead of loop, consider just calculating adding new weight to the totalWeight. Change to:
```
totalWeight+=_weight;
```

##### Saved
This saves ~400 (if we take derivativeCount = 3).
#
### G-7. Use unchecked blocks for subtractions where underflow is impossible
##### Description
In Solidity 0.8+, there’s a default overflow and underflow check on unsigned integers. When an overflow or underflow isn’t possible (after require or if-statement), some gas can be saved by using [unchecked blocks](https://docs.soliditylang.org/en/v0.8.17/control-structures.html#checked-or-unchecked-arithmetic).
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L201

##### Saved
This saves ~35.  
#
### G-8. Implement a function for removing a derivative
##### Description
Consider implementing a function for removing a derivative from the ```derivatives``` mapping. Some functions will continue to call derivative contracts which are not longer used (when set weight = 0, and all assets are withdrawn). The reason is that many functions loop through the ```derivativeCount``` iterations, i.e. all derivatives.
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71-L75
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84-L86
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113-L116
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140-L141
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147-L148
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171-L172
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191-L192

#
### G-9. Do not call ```balance()``` functions twice
##### Description
The value returned by ```derivatives[i].balance()``` can be cached.
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L73-L74
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L141-L142 

#
### G-10. Some expressions can be transformed
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173-L174

##### Recommendation
Can be changed to:
```
uint256 minOut = ((ethPerDerivative(_amount) * _amount * (10 ** 18 - maxSlippage)) / 10 ** 36;
```
and to:
```
uint256 minOut = (((rethPerEth * msg.value) * (10 ** 18 - maxSlippage)) / 10 ** 36);
```

#
### G-11. Do not multiply and divide by the same number
##### Instances
- [```else return (poolPrice() * 10 ** 18) / (10 ** 18);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215) 

##### Recommendation
Change to ```else return poolPrice(); //(poolPrice() * 10 ** 18) / (10 ** 18)```

#
### G-12. Use uint256(1) and uint256(2) for pausing
##### Description
Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.  
##### Instances
- [```pauseStaking = _pause;```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L233)
- [```pauseUnstaking = _pause;```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L242) 

##### Recommendation
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.