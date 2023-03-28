## Pausing functionality can be done with just one function

There are currently separate functions for pausing and unpausing

```solidity
   function setPauseStaking(bool _pause) external onlyOwner {
        pauseStaking = _pause;
        emit StakingPaused(pauseStaking);
    }

    /**
        @notice - Owner only function that enables/disables the unstake function
        @param _pause - true disables unstaking / false enables unstaking
    */
    function setPauseUnstaking(bool _pause) external onlyOwner {
        pauseUnstaking = _pause;
        emit UnstakingPaused(pauseUnstaking);
    }
```

This is unnecessary as one function is enough, the `stakingPaused` variable alone can be used to check were staking is paused or unpaused.


## Validate minimum and maximum values when setting slippage

```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

Currently there are no restrictions on the amount of slippage that can be set. To prevent unintended effects, the input should be validated for acceptable minimum and maximum slippage.

## Validate that `totalWeight` should never be 0

```soldity
  function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
        weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit WeightChange(_derivativeIndex, _weight);
    }
```

When the weights are being set or adjusted, it would be good practice to ensure the `totalWeight` isn't 0 in a scenario where either by malice or mistake, all weights are set to 0.

## Validate that the `_contractAddress` of a derivative cannot be set to 0

```solidity
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

When adding a derivative, a check should be done to prevent `_contractAddress` being set to 0 by mistake.

## Weight calculation can result in some derivatives failing to get a proportional stake

```solidity
uint256 ethAmount = (msg.value * weight) / totalWeight;
```

If `msg.value` is a low value and a derivative has low weight, there are instances where the `ethAmount` will be zero. 

A suggestion would be to enforce a minimum weight to total weight ratio for derivatives.