### G[1] Use arithmetic operating instead using loop for calculating totalWeight adjustment 
[Link to github permalink](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175)
```
function adjustWeight(
    uint256 _derivativeIndex,
    uint256 _weight
) external onlyOwner {
+    uint256 oldWeight = weights[_derivativeIndex];
+    weights[_derivativeIndex] = _weight;
+    totalWeight = totalWeight - oldWeight + _weight;

-    weights[_derivativeIndex] = _weight;
-    uint256 localTotalWeight = 0;
-    for (uint256 i = 0; i < derivativeCount; i++)
-       localTotalWeight += weights[i];
-    totalWeight = localTotalWeight;
    emit WeightChange(_derivativeIndex, _weight);

}
```

The code above shows a function called `adjustWeight` that allows the owner to adjust the weight of a derivative. The function has been optimized to reduce gas consumption by eliminating the loop and instead, tracking the old weight, updating the new weight, and recalculating the total weight with a simple arithmetic operation.

This new code adds a variable `oldWeight` to store the original weight of the derivative before updating it. Then, the function updates the derivative's weight with the new weight provided by the owner and recalculates the total weight by subtracting the old weight and adding the new weight. This eliminates the need for a loop that calculates the total weight of all derivatives.

## G[2] Use arithmetic operating instead using loop for new totalWeight calculating
[Link to github permalink](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182-L195)
```
function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;

       // @remind gas/extra calcualtion
+      totalWeight = totalWeight + _weight;

-      uint256 localTotalWeight = 0;
-       for (uint256 i = 0; i < derivativeCount; i++)
-           localTotalWeight += weights[i];
-       totalWeight = localTotalWeight;
        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    }
```
Can be easily update `totalWeight = totalWeight + _weight;` since ` x = x + y` take less gas `x+=y`.
also there is no need of `for loop` to recalculate weight.

## G[3] Use arithmetic operating instead using loop for new totalWeight calculating
[Link to github permalink](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138-L155)
```diff
 function rebalanceToWeights() external onlyOwner {
        uint256 ethAmountBefore = address(this).balance;
        for (uint i = 0; i < derivativeCount; i++) {
            if (derivatives[i].balance() > 0)
                derivatives[i].withdraw(derivatives[i].balance());
        }
        uint256 ethAmountAfter = address(this).balance;
        uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
        // @remind
+       if ethAmountToRebalance > 0{
            for (uint i = 0; i < derivativeCount; i++) {
-                if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
+                if (weights[i] == 0) continue;
                uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                    totalWeight;
                // Price will change due to slippage
                derivatives[i].deposit{value: ethAmount}();
            }
+        }
        emit Rebalanced();
    }
```
Here, by adding `if ethAmountToRebalance > 0` condition, we can save a lot of gas by escaping from loop if `ethAmountToRebalance = 0` since `ethAmountToRebalance` been checked for every iteration.


