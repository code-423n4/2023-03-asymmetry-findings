# Gas Optmization Findings

## Replace i++ in for loop
Could be replace with ++i or unchecked math block
```solidity
// option1
for (uint256 i = 0; i < derivativeCount; ++i)
{
    // ...
}
```
```solidity
// option2
for (uint256 i = 0; i < derivativeCount;)
{
    // ...
    unchecked{
        ++i;
    }
}
```
Occurrences
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191

## totalWeight calculation could be further optimized
The `adjustWeight` and `addDerivative` function will result in `totalWeight` change. But instead of re-calcualting it by for loop iterating, the `totalWeight` could be updated by adding the weight of new derivative (in `addDerivative` case) or the difference between new and old weight (in `adjustWeight` case). It could save multiple storage reads.
```solidity
function adjustWeight(
    uint256 _derivativeIndex,
    uint256 _weight
) external onlyOwner {
    uint256 originalWeight = weights[_derivativeIndex];
    totalWeight = totalWeight + _weight - originalWeight;
    weights[_derivativeIndex] = _weight;
    emit WeightChange(_derivativeIndex, _weight);
}
```
```solidity
function addDerivative(
    address _contractAddress,
    uint256 _weight
) external onlyOwner {
    derivatives[derivativeCount] = IDerivative(_contractAddress);
    weights[derivativeCount] = _weight;
    derivativeCount++;

    totalWeight += _weight;
    emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
}
```