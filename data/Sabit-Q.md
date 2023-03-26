
Redundant import: The contract imports the OwnableUpgradeable.sol file two times.  
Remove one.
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/mocks/DerivativeMock.sol#L4

In the withdrawAll() function, you can use payable(msg.sender) and transfer instead of call.
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/mocks/DerivativeMock.sol#L38