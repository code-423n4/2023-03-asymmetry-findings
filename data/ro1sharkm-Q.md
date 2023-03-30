## The contract uses 'abi.encodePacked' instead of 'abi.encode' in some parts of the codebase
This can potentially lead to issues with the compatibility of the contract with future versions of Solidity.
It is recommended to replace 'abi.encodePacked' with 'abi.encode' in the affected parts
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L162
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L191
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L233

## Potential Division by Zero
The '_safEthTotalSupply' variable in the 'unstake' function used as the divisor in a division operation could be set to zero, which would result in a runtime error.
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L115-L116
To mitigate appropriate checks should be implemented to ensure that '_safEthTotalSupply' variable is not set to 0.

## Floating pragma directive used in the contract
The contract contains floating pragma directives, which can lead to potential compatibility issues with future compiler versions.
It is recommended to add a fixed pragma directive at the beginning of the contract specifying a specific compiler version.


