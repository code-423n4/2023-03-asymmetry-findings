SafEth.sol

1. `adjustWeight()` -> You don't need to use a loop to calculate the new totalWeight. Instead you only need to calculate the difference between the current weight of `_derivativeIndex` and the new value, and then add that to the existing totalWeight. Make a signed int256 that equals `weights[derivativeIndex] - _weight. Then do totalWeight -= that int. If the new weight is greater then you'll be subtracting a negative and the total weight will be greater only by the difference in weights. Example: original weight = 1e18 and new weight = 2e18. The new total weight should be 1e18 greater than before. totalWeight -= (1e18 - 2e18) => totalWeight + 1e18. This prevents the need for the loop which requires O(n) steps and replaces it with an O(1) computation of a only 2 SLOAD's and 1 SSTORE. The same can be done in `addDerivative`

2. `unstake()` and `stake()` -> Increase loop efficiency by caching the derivativeCount value in memory, pre-incrementing the index, and using an unchecked block. index `i` is safe since it cannot overflow and using a local derivativeCount prevents O(i) SLOADS from needing to occur to read the state every loop 
```
uint localDerivativeCount = derivativeCount;
for(uint i = 0; i < localDerivativeCount;) {
//do the stuff in the loop
unchecked {
++i;
}
}//end of loop

3. Replace (10 ** 18) with constant `1e18` or relevant value in scientific notation. Using ** uses an unnecesarry `exp` operation. Saves >=60 gas every time is used
lines 75, 82, 83, 102, 98, 55, 54

4. line 80 -> Set preDeposit price = 1e18 at initialization and only set it to the equation at line 83 if totalSupply != 0. Saves a jump by removing one conditional branch

--- SafEthStorage.sol---
Replace pauseStaking and pauseUnstaking with uint = 1, and when pauses/unpaused change to 2. Using a bool means an SSTORE that turns a zero-slot into a non-zero slot which costs 21,000 gas every time. Changing a non-zero slot to a non-zero slot only costs 100 gas when pausing/unpausing occurs.


---SfrxEth.sol---
1. Re-use the exponent optimization from above. In `withdraw` saves 180 gas from 10**18 being used 3 times. Is also used on line 113 and 115

2. 