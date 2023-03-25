Defining initial values for variables when declaring them in a contract like in the code below does not work for upgradeable contracts.
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L38

Refer to explanation below:

https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#avoid-initial-values-in-field-declarations

Also, one should not leave the implementation contract uninitialized. None of the implementation contracts in the code base contains the code recommended by OpenZeppelin below, or an empty constructor with the initializer modifier.

/// @custom:oz-upgrades-unsafe-allow constructor
constructor() {
    _disableInitializers();
}
Refer to the link below:

https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#initializing_the_implementation_contract