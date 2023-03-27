## 1. Reduce repetition of code
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L170-L173
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L190-L193
The mentioned code fragments are repeated, instead put it in a method. (Don't repeat the code, Write Code Once)

###### Recommended Solution
1 . Create a new function:
```
function recalculateTotalWeight() private {
        uint256 localTotalWeight = 0;
        for (uint256 i; i < derivativeCount; ++i)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
}
```
2 . Call the function wherever you need, for example:
```
function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;

        recalculateTotalWeight(); // <<--------- Call the function ---------------

        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    }
```

```
function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
        weights[_derivativeIndex] = _weight;
        
        recalculateTotalWeight(); // <<--------- Call the function -------------

        emit WeightChange(_derivativeIndex, _weight);
    }
```