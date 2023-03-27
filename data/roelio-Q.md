Overall code is simple and self-explanatory. The codebase is small and to the point. Modern solidity is applied.

# Low severity

## Wrong price of the lido derivative
* Problem: https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L87 returns the amount of stETH for one wstETH. And it should return the amount of ETH for one wstETH.

* Solution: Use the curve or chainlink oracle to determine the amount of ETH per wstETH


## Adjusting weights of non-existing derivatives
* Problem: It is possible to set the weight of a derivative that doesnt exist https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L169. Though it doesnt cause issues its probably good to revert as feedback.

* Solution: require the derivative to exist


## Slippage should be limited

* Problem: Both https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L52 and https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L49 https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L59 should probably be restricted to not allow extreme slippage which could be used to do a malicious arbitrage. 

 * Solution: Have an upper bound to the slippage. A one for all solution could be done in https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L206


## Owner can hijack funds by adding derivative and adjusting weights.

* Problem: function https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182 allows the owner to add a malicious derivative and could then change weights to move all eth to this contract and steal the funds.

* Solution: In this case the owner should ideally be behind a timelock, but maybe this is not okay for pausing the staking so potentially add an additional role for immediate actions and put the rest behind timelock.

## Overflow in localTotalWeight calculation

* Problem: In the scenario where the weights add up to be more than the uint256 can hold the transaction will revert (https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L192, https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L172)

* Solution: Use a smaller uint8 or similar for storing the weights.


