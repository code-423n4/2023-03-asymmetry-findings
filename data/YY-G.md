# Potential gas optimization in the `addDerivative` function which related to the loop used to calculate the totalWeight of all derivatives.

## Affected Code
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-L195

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

## Description and Impact

The issue is related to the loop used to calculate the totalWeight of all derivatives. 
Every time a new derivative is added, the loop iterates over all the derivatives in the weights array to calculate the totalWeight. As the number of derivatives increases, this loop becomes more expensive to execute, which can result in higher gas costs. 



