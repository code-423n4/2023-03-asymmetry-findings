## Unnecessary variables creation
The contract `SafeEth.sol` contains functions **stake**, **unstake** and **rebalanceTokens**. Inside these functions is a for loop over `derivates` mapping. In each iteration of the loop, new local variables are created. 

In **stake** function there are 4 unnecessary variable creations: `uint256 weight`, `uint256 ethAmount`, `uint256 depositAmount`, `uint derivativeReceivedEthValue`

In **unstake** function there is 1 unnecessary variable creation: `uint256 derivativeAmount`

In **rebalanceTokens** function there is 1 unnecessary variable creation: `uint256 ethAmount`

Creating new variables inside the loops costs additional gas and should be avoided. Move the variable creation before the loop cycle and leave only the assignment inside the loop.

## Redundant variable assignment
In the contract `SafeEth` line #87 is a condition for skipping the loop iteration when `weight == 0`.
In line #86 is the variable `derivative` creation and assignment. This assignment is redundant if the previous condition is *True*. Switch the lines and check the weight before the assignment to safe some gas.

##