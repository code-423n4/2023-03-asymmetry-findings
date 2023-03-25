G1. adjustWeight() can save gas by eliminating the for loop:
```diff
 function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
+       if(_derivativeIndex >= derivativeCount) CannotAdjustWeightForNonExistingDerivative();
+       uint256  localTotalWeight = totalWeight-weights[_derivativeIndex];
        weights[_derivativeIndex] = _weight;
+       uint256  localTotalWeight += _weight; 
-        uint256 localTotalWeight = 0;
-        for (uint256 i = 0; i < derivativeCount; i++)
-            localTotalWeight += weights[i];
         totalWeight = localTotalWeight;
         

        emit WeightChange(_derivativeIndex, _weight);
    }
```