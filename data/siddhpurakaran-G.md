1.Avoid looping through all derivatives and doing arithmetic operations inside loops,  when the same can be achieved by a single local variable.
->A.In addDerivative function find totalWeight just by adding difference of old and new value to totalWeight.

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
->Convert above loop to this code.
        totalWeight = totalWeight + (oldWeight - newWeight);
->B.In the adjustWeight function the same thing can be applied.

2.Move require check immediately after the local variable initialization in stake function.

L-85        uint256 weight = weights[i];
L-86       	IDerivative derivative = derivatives[i];
L-87        if (weight == 0) continue;

->Here we have to put weight==0 check before derivative variable declaration.

L-85        uint256 weight = weights[i];
L-86        if (weight == 0) continue;
L-87       	IDerivative derivative = derivatives[i];
