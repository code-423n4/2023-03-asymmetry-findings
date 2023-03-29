1. Lack of input validation in setPauseStaking() and setPauseUnstaking()
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L232-L244

When using the ```setPauseStaking``` and ```setPauseUnstaking``` function, it's better to check if the input parameters are different than the global ones. So unnecessary changes will be discarded and confusing events won't be out:

```
function setPauseStaking(bool _pause) external onlyOwner {
        require(_pause != pauseStaking, " pauseStaking unchanged!");
        pauseStaking = _pause;
        emit StakingPaused(pauseStaking);
    }

    /**
        @notice - Owner only function that enables/disables the unstake function
        @param _pause - true disables unstaking / false enables unstaking
    */
    function setPauseUnstaking(bool _pause) external onlyOwner {
        require(_pause != pauseUnstaking, "pauseUnstaking unchanged!");
        pauseUnstaking = _pause;
        emit UnstakingPaused(pauseUnstaking);
    }

```

2. lack of validation for the existence of derivative in adjustWeight()
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L165-L175
The existence of derivatives should be checked before adjusting the weight, otherwise if weight is set for an unexisting derivative, will result in the wrong totalWeight and cause panic in stake(), unstake() function.
```
function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
        require(__derivativeIndex < derivativeCount, "derivative does not exist!")
        weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit WeightChange(_derivativeIndex, _weight);
    }
```
Lack of input checks on setMinAmount() and setMaxAmount()
```setMinAmount(),setMaxAmount()``` don't have any checks if minAmount and maxAmount are consistent. If they are not, then ```stake()``` will always revert. 
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L214-L217
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L223-L226
Let's assume that current minAmount = 10 * 10 ** 18 (which is 10 ETH) and maxAmount = 30* 10 ** 18(30 ETH), then the owner forgot to check the values of maxAmount before setting minAmount to 32 ETH. Then the ```stake``` function will always revert due to the check that
```
require(msg.value >= minAmount, "amount too low");
require(msg.value <= maxAmount, "amount too high");
```

Add checkes in both set functions to make sure minAmount<= maxAmount
```
    function setMinAmount(uint256 _minAmount) external onlyOwner {
        require(_minAmount <= maxAmount, "_minAmount is larger than maxAmount!");
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }
```
```
 function setMaxAmount(uint256 _maxAmount) external onlyOwner {
        require(_maxAmount >= minAmount, "_maxAmount is less than minAmount!");
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
    }
```

