Confirm the derivative address to avoid incorrectly setting weight.
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175

    function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight,
        address _checkDerivative
    ) external onlyOwner {
        require(_checkDerivative == address(derivatives[_derivativeIndex]), "wrong derivative");
        weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        for (uint i = 0; i < derivativeCount; i++) {
            uint256 weight = weights[i];
            if (weight == 0) continue;
            uint256 ethAmount = (minAmount * weight) / totalWeight;
            require(ethAmount > 0, "weight too low");
        }
        emit WeightChange(_derivativeIndex, _weight);
    }
```
Adding and removing functions can reduce gas costs and minimize unknown risks.
```
    function removeDerivative(
        uint256 _derivativeIndex,
        address _checkDerivative
    ) external onlyOwner {
        require(_checkDerivative != address(0), "invalid address");
        require(_checkDerivative == address(derivatives[_derivativeIndex]), "wrong derivative");
        adjustWeight(_derivativeIndex, 0,_checkDerivative);
        rebalanceToWeights();
        --derivativeCount;
        derivatives[_derivativeIndex] = derivatives[derivativeCount];
        weights[_derivativeIndex] = weights[derivativeCount];
    }
```
Lower gas, reduced risk, easier to understand.
```
struct Derivative {
    IDerivative derivative;
    uint96 weight;
}
mapping(uint256 => Derivative) public derivatives;
```

totalEth and totalEthAmount are not always equal,  Whether returning or depositing, we should handle it.
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L83-L96
        uint256 totalEth = msg.value;
        uint256 totalEthAmount;
        for (uint i = 0; i < derivativeCount; i++) {
            uint256 weight = weights[i];
            IDerivative derivative = derivatives[i];
            if (weight == 0) continue;
            uint256 ethAmount = (totalEth * weight) / totalWeight;
            totalEthAmount += ethAmount;
            // This is slightly less than ethAmount because slippage
            uint256 depositAmount = derivative.deposit{value: ethAmount}();
            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
                depositAmount
            ) * depositAmount) / 10 ** 18;
            totalStakeValueEth += derivativeReceivedEthValue;
        }
        uint256 remaining = totalEth - totalEthAmount; // They are not always equal,  Whether returning or depositing, we should handle it.

```

Convert uint to uint256