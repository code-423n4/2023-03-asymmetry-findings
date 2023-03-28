## Add input validation to `unstake`
Consider adding a `require` statement to for zero amount being unstaked. This makes unstaking revert with a sensible message instead of an overflow/underflow error. It also prevents possible unwanted behavior in future scenarios.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108
