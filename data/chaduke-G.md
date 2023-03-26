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

G2. We can save gas for addDerivative() by eliminating the for loop.
```diff
function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;

-        uint256 localTotalWeight = 0;
-        for (uint256 i = 0; i < derivativeCount; i++)
-            localTotalWeight += weights[i];
-        totalWeight = localTotalWeight;
+        totalWeight = totalWeight+_weight;
        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    }
```
G3. One can simply return poolPrice() here:
```diff
 function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        if (poolCanDeposit(_amount))
            return
                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
-        else return (poolPrice() * 10 ** 18) / (10 ** 18);
+        else return poolPrice();
    }
```
