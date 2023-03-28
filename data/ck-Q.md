## Pausing functionality can be done with just one function

There are currently separate functions for pusing and unpausing

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

## Weight calculation can result in some derivatives failing to get a proportional stake

```solidity
uint256 ethAmount = (msg.value * weight) / totalWeight;
```

If `msg.value` is a low value and a derivative has low weight, there are instances where the `ethAmount` will be zero. 

A suggestion would be to enforce a minimum weight to total weight ratio for derivatives.