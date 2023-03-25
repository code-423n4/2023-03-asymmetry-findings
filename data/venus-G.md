**1. Gas Report**
*minAmount* and *maxAmount* variables are declared as public, initialized in initialize function(hard coded) and never changed in anywhere else in the code. Therefore, they should have been declared as constant because constant variables occupies less storage and cost lower Gas.
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L54-L55 

**2. Gas Report**
Function below in the SafEth contract unnecessarily sums each weight from the beginning to the end. This causes high Gas consumptions. 

    function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;

        uint256 localTotalWeight;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    }

It should not calculate the total weight by calling all the weights. Instead, it needs to  add the new weight to total sum or if the contract already exists it will just apply the difference.