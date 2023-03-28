# Dynamic updating of `totalWeight`
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L169-L173
```
        weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L187-L193
```
        weights[derivativeCount] = _weight;
        derivativeCount++;

        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
```
It seems like current code assume that `derivativeCount` is not big.
But in case of `derivativeCount` is big, we need to optimize these for loops.
Like this:
```
// in adjustWeight(L169-L173)
        totalWeight -= weights[_derivativeIndex];
        weights[_derivativeIndex] = _weight;
        totalWeight += weights[_derivativeIndex];

// in addDerivative(L187-L193)
        weights[derivativeCount] = _weight;
        derivativeCount++;
        totalWeight += weights[_derivativeIndex];
```

# We can move `ethAmountToRebalance = 0` before the for loop
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L147-L153
```
        for (uint i = 0; i < derivativeCount; i++) {
            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                totalWeight;
            // Price will change due to slippage
            derivatives[i].deposit{value: ethAmount}();
        }
```
Optimized Code by avoiding multiple checking of `ethAmountToRebalance != 0`.
```
        if (ethAmountToRebalance != 0) {
            for (uint i = 0; i < derivativeCount; i++) {
                if (weights[i] == 0) continue;
                uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                    totalWeight;
                // Price will change due to slippage
                derivatives[i].deposit{value: ethAmount}();
            }
        }
```

# Too many use of same integer `10 ** 18` without declaring it as a constant.
In 4 solidity files which are in the scope, we can see totally 20 times of `10 ** 18`.
If we declare a constant variable of this, we can reduce the gas.
