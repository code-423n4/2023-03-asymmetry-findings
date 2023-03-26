## Decription: insufficient event emmission.
Function adjustWeight() in the safEth.sol make changes to totalWeight stateVariable and did not emit the current state, as best practice, it is adviceable to emit changes made to state for ofchain usage.

## context: [SafEth.sol#L173](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L173)
# Recommendation:
It is recommended to add the new totalWeight to the event emmited in adjustWeight() for example:
`event WeightChange(uint indexed index, uint weight);`
 should be replaced with
`event WeightChange(uint indexed index, uint indexed weight, uint indexed _NewtotalWeight);`