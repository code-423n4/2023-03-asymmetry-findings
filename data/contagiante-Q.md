## Part 1

## Title: Use safeTransferOwnership instead of transferOwnership function

## Lines of Code

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L18

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L9

## Impact

transferOwnership function is used to change Ownership from OwnableUpgradeable.sol.

Use a 2 structure transferOwnership which is safer.
safeTransferOwnership, use it is more secure due to 2-stage ownership transfer.

## Recommended Mitigation Steps
Use Ownable2StepUpgradeable.sol
Ownable2StepUpgradeable.sol https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol

## Part 2

## Title: Sanity Checks for major function

## Lines of Code

1)https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-L195

2)https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202-L208

3)https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214-L217

4)https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223-L226

5)https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232-L235

6)https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241-L244

## Recommended Mitigation Steps

1)-Check for _contractAddress variable not to be equal to dead wallet
   -Check for _weight variable not equal to 0

2)Check for _slippage variable not to be equal to 0 and should be less than 100% so that the owner can not set a slippage number that can lead to the destruction of the protocol.

3)Check for _minAmount variable not to be equal to 0 and less than _maxAmount

4)Check for _maxAmount variable not equal to 0 and greater than _minAmount
