### G[1] Use arithmetic operating instead using loop for calculating totalWeight adjustment 
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
