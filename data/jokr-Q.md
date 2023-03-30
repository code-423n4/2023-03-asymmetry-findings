```
function adjustWeight(
    uint256 _derivativeIndex,
    uint256 _weight
) external onlyOwner {
    weights[_derivativeIndex] = _weight;
    uint256 localTotalWeight = 0;
    for (uint256 i = 0; i < derivativeCount; i++)
        localTotalWeight += weights[i];
    totalWeight = localTotalWeight;
    emit WeightChange(_derivativeIndex, _weight);
}
```

```
function addDerivative(
    address _contractAddress,
    uint256 _weight
) external onlyOwner {
    derivatives[derivativeCount] = IDerivative(_contractAddress);
    weights[derivativeCount] = _weight;
    derivativeCount++;

    uint256 localTotalWeight = 0;
    for (uint256 i = 0; i < derivativeCount; i++)
        localTotalWeight += weights[i];
    totalWeight = localTotalWeight;
    emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
}
```

In above two functions in `SafEth.sol`, the totalWeight calculation is done by looping all weights.
Instead totalWeight can be updated by using the currently updated weight

It can be done like below. Same applies for other function too
```
function adjustWeight(
    uint256 _derivativeIndex,
    uint256 _weight
) external onlyOwner {

    uint256 prevWeight =  weights[_derivativeIndex];

    if (_weight > prevWeight){
        totalWeight +=_weight - prevWeight;
    }
    else {
        // will revert on overflow. So no issue
        totalWeight -= prevWeight - _weight;
    }

    weights[_derivativeIndex] = _weight;
    
    emit WeightChange(_derivativeIndex, _weight);
}

```