# [G-01] Cache the balance retrieval to avoid retrieving it twice.
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L73-L74
Before 532939 After 524794
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L141-L142
Before 729728 722422
# [G-02]  Maintain running totals instead of looping for adjustWeights and addDerivative
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L165-L175
Example update
```
function addDerivative(address _contractAddress, uint256 _weight) external onlyOwner {
    derivatives[derivativeCount] = IDerivative(_contractAddress);
    weights[derivativeCount] = _weight;
    derivativeCount++;
		  
    totalWeight += _weight; // Maintain a running total of weights
    emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
}
```
Before 102462 After 99501
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182-L195
Example update
```
function adjustWeight(uint256 _derivativeIndex, uint256 _newWeight) external onlyOwner {
    require(_derivativeIndex < derivativeCount, "Invalid derivative index");
		  
    uint256 _oldWeight = weights[_derivativeIndex];
    weights[_derivativeIndex] = _newWeight;
		  
    // Update totalWeight using the difference between the new and old weights
    totalWeight = totalWeight - _oldWeight + _newWeight;
		  
    emit WeightChange(_derivativeIndex, _newWeight);
    }
```
Before 46507 after 41294