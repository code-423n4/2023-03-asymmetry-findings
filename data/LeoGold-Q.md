# 1
## Decription: insufficient event emmission.
Function adjustWeight() in the safEth.sol make changes to totalWeight stateVariable and did not emit the current state, as best practice, it is adviceable to emit changes made to state for ofchain usage.

## context: [SafEth.sol#L173](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L173)
# Recommendation:
It is recommended to add the new totalWeight to the event emmited in adjustWeight() for example:
`event WeightChange(uint indexed index, uint weight);`
 should be replaced with
`event WeightChange(uint indexed index, uint indexed weight, uint indexed _NewtotalWeight);`

# 2

## Description: one step ownership transfer can cause total loss of control of SafEth.sol contract
In a case where the owner of the safEth.sol contract intend to change the owner to an another user, there is possibility that if a wrong address is passed to transferOwnnership, the control of the SafEth.sol lose total control of the wrong address and there is no way to get the control back if the address cannot be traced to a known person.

##Re commendation.
it is Adviceable to implement the Ownable2StepUpgradeable.sol from openzeppeline to give room for the new owner to accept the ownership before control is being transferred check [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) for details of the 2 step verification of transfer ownership.

# 3
## Description: Use a more recent version of Solidity
For security, it is best practice to use the latest Solidity version.
For the security fix list in the versions;
## context All contracts in scope.

## Recommendation:
Old version of Solidity is used (0.8.13), newer version can be used (0.8.19).

# 4.
## Description: Lock pragmas to specific compiler version.
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.
[swc-103](https://swcregistry.io/docs/SWC-103)
## Context All contracts in scope.
## Recommmendation.
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.
[smart contract best practice](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/).

# 5.
## Description. function `setMaxAmount() and setMinAmount` can be set to conflict eachother.
Owner can set minAmount that conflict with maximum Amount.

## context: [SafEth.sol L214-L226](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L214-L226).

## Recommendation.
it best to confirm the parameter set by Owner to check that setMaxAmount arguement is not less than the minimum amount set and the parameter set for setMinAmount is not greater than maxAmount set.

# 6. 
## Description:
inconsistency with the `unstake()` function, users are unpredictable and can try to withdraw as low as 1wei, 2 wei etc and for a very little amount, it shows that while you caan call the unstake function successfully with passing 0, 1, 2 and 3 as value, user SafEth reduced in value but received 0 ethers in return for the amount deducted in safEth. And also passing 5 and 6 as a value in `unstake function` get reverted at the the point of exchanging on lido protocol calling the exchange function. (this was written in vyper).
## context [SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L108).

## Recommendation.
it is recommended to add minmum amount that can be unstake to correct the inconsistency happening with the `unstake functions`.


