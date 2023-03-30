# [G‑01] Refactor to avoid loop operation over `weights` and `derivatives` state variables

If `ethAmountToRebalance` is equal to 0, the function `rebalanceToWeights` should stop and return 0 immediately.

However, in the current code, even if `ethAmountToRebalance` is equal to 0, it loops through each item in `weights` and checks if the weight is equal to 0. If the weight is not equal to 0, then it performs calculations and external calls to deposit to the derivatives contract.

By refactoring the code to stop immediately when ethAmountToRebalance is equal to 0, we can save on costs of SLOADs and external calls to derivatives contracts.

Instance:

https://github.com/code-423n4/2023-03-asymmetry/blob/0e826f307866f3b9e737a3fd7b860e1df2dd7e71/contracts/SafEth/SafEth.sol#L147-L153