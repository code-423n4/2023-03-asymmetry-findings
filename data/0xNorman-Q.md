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