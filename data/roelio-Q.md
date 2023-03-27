# Slippage should be limited

* Problem: Both https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L52 and https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L49 https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L59 should probably be restricted to not allow extreme slippage which could be used to do a malicious arbitrage. 

 * Solution: Have an upper bound to the slippage. A one for all solution could be done in https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L206


# Owner can hijack funds by adding derivative and adjusting weights.

* Problem: function https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182 allows the owner to add a malicious derivative and could then change weights to move all eth to this contract and steal the funds.

* Solution: In this case the owner should ideally be behind a timelock, but maybe this is not okay for pausing the staking so potentially add an additional role for immediate actions and put the rest behind timelock.

# Overflow in sum calculation of addDerivative

* Problem: In the unlike scenario where the weights add up to be more then the uint256 can hold the transaction will revert
* Solution: Use a smaller uint8 or similar for storing the weights.