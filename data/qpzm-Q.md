I submitted the gas issue by mistake, so re-submit in QA report🙏.

## Cache the storage variable in for loop.
Reduce the number of sload.
```solidity
// cache storage variable in stack.
uint256 _derivativeCount = derivativeCount;
for (uint256 i = 0; i < _derivativeCount; i++) {
// ...
}
```

- https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71
- https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L84
- https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113
- https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140
- https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L147
- https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L171
- https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L191


## `adjustWeight`: Remove the for loop as below.
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L169

```solidity
function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
        uint256 beforeWeight = weights[_derivativeIndex];
        weights[_derivativeIndex] = _weight;
        totalWeight = totalWeight - beforeWeight + _weight;
        emit WeightChange(_derivativeIndex, _weight);
    }
```

## `addDerivative`: Remove the for loop as below.
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L190-L193

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
