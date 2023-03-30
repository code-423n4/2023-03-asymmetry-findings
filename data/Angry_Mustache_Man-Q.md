# [L-01] Floating pragma
It is always best to lock pragma rather than having a floating pragma version.
### Context
https://swcregistry.io/docs/SWC-103
### Occurence
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L2
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L2
# [L-02] Use Ownable2StepUpgradeable instead of OwnableUpgradeable contract
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L5
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L13
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L6
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L9
transferOwnership function is used to change Ownership from Ownable.sol.
Use a 2 structure transferOwnership which is safer. safeTransferOwnership, use is more secure due to 2-stage ownership transfer.
# [L-03] Precision Loss 
Calculations done in contracts at multiple location lead to precision losses. Some instances are 
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L81
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L98
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L116
# [L-04] No specific recovery mechanism 
No specific recovery mechanism in SafEth ,if the owner gets compromised which might in turn lead to critical issues.
# [N-01] Function doesn't use the parameter 
### Context
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L111
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86