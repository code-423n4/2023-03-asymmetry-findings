# No Verification Check inside addDerviative function to check it implements IDervative Interface.
All the Derivative Contract implements <em>`IDerivative`</em> to add the new Derivative Contract to Protocol The Function inside SafeEth 
```SafEth:addDerivative``` is used to add new Derivative contracts. 

This Function have a access modifier of ``owner:onlyOwner()``. which is fine , we understand that whenever new derivative will be added to the protocol all the check will be verified off-chain or manually by protocol owner or tech team.
This function do parse the Address of derivative `` derivatives[derivativeCount] = IDerivative(_contractAddress)`` but it will not revert if the given contract does not implements ``IDerivative`` interface.
If such Derivative contract added to the protocol the ``SafEth:stake`` and ``SafEth:unstake`` methods  will not work as intended because the call to  `ethPerDerivative(uint256)` and ``withdraw(uint256)`` function from derivative contracts. this will result into an unintended behaviour of protocol.
#### Recommendation 
we recommend to use ``ERC165Checker`` for contract address  (which helps to check weather the given contract implements given interfaceId) and add require statement or modifier to check it. 
