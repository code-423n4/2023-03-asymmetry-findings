# Add validation of _derivativeIndex < derivativeCount in SafETH.adjustWeight

In [SafETH.adjustWeight](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L177) it doesn't make sense if _derivativeIndex >= derivativeCount. It's better to add such check in case of contract owner call this function with wrong params accidentally.

```solidity
function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
  require(_derivativeIndex < derivativeCount, "Invalid _derivativeIndex");
  ...
}
```