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
it is Adviceable to implement the Ownable2StepUpgradeable.sol from openzeppeline to give room for the new owner to accept the ownership before control is being transferred check [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) for details of the 2 step verification of transfer ownership 