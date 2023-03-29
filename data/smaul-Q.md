# Optimize the pausing into an single operation.
[SafEth.sol#L64](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L64) , [SafEth.sol#L109](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L109)

Change `pausestaking` & `pauseunstaking` in a single variable, and use it to pause both staking & unstaking.

Once something bad happens to the protocol you should pause all operation in a single function call. 
Delete `pausestaking` & `pauseunstaking` and use `pause` and change `pausestaking` & `pauseunstaking` to `pause` all over the code base.