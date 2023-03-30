QA report for Pause/Unpause logic (including cases of paused derivatives).

#1:
It appears that handling of paused any or all of the derivatives is not currently built into `SafEth` contract and the outcomes are not clear.
Variable `derivativeCount` should be expanded and include special handling of paused derivatives and propagate changes to adjusting & rebalancing weights logic.
Simply relying on derivativeCount in `SafEth.stake` and `SafEth.unstake` that include paused derivative may lead to possible financial losses for `SafEth` holder in the event of underlying derivates being paused for various concerns (including security). 
Specifically, it is unclear how the protocol handles attempts to invoke `stake` function that accepts `ETH` and tries to convert it into a derivative that is not active. 

One of the possible remediations is to offer an implementation of voluntary exit for a user which may include paying some sort of redemption fee.
Another is to build a separate set of calculations to adjust and/or rebalance derivative/s weight capturing the derivative’s last valid/known key points. 

#2
It is unclear why pausing/unpausing logic is only applied to several functions rather than to the entire contract with specific modifiers. Suggesting using open zeppelin libraries to handle pause/unpause logic, e.g. `PausableUpgradeable` lib.

The current state of the contract:
- erroneously allows pause already paused stake and unstake functions;
- erroneously allows unpause already unpaused stake and unstake functions;
- erroneously doesn’t restrict transfer ownership when paused;
