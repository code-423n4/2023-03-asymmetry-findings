### 1. Unnecessary loop in addDerivative()

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182

addDerivative() function doesn’t need to  have a for loop for updating totalWeight.
totalWeight could just be increased by _weight.

### 2. Unnecessary loop in rebalanceToWeights()

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138

In the rebalanceToWeights() function, the second loop checks ethAmountToRebalance == 0 every time.
If ethAmountToRebalance is 0, no need to have the second loop.
So this check could be moved to before the second loop.