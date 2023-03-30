- use `unchecked {++i;}` to increment `i` in for-loops to avoid SafeMath check every iterations, since we know this never overflows. For example, `for (uint256 i = 0; i < derivativeCount; i++) { … }` can be changed to `for (uint256 i; i < derivativeCount;) { … ; unchecked{++i;}}

- `adjustWeight` now loops through every stored `weights` to calculate the new sum. However, you can easily update the total weight as:
```
function adjustWeight(uint _derivativeIndex, uint _weight) external onlyOwner {
  uint oldWeight = weights[_derivativeIndex]; 
  weights[_derivativeIndex] = _weight; // update value
  totalWeight = totalWeight + _weight - oldWeight;
  emit ...
}
```
This uses only 2 SLOADs, compared to linear number of SLOADs.

- Same logic as above total weight update can be optimized in `addDerivative` function.

- `require(pauseUnstaking == false, "unstaking is paused”);` can be changed to `require(!pauseUnstaking, "unstaking is paused”);`