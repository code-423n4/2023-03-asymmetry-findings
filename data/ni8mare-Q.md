## Low Risk Issues

In the contract [SafEth](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol):

a. In `addDerivative` function: 
Check for zero address for the param `_contractAddress`
`require(_contractAddress != address(0))`

b. In `adjustWeight` function:
Check that the contract exists at the `_derivativeIndex` param. Otherwise, the weight could be adjusted for a derivative that does not exist.
`require(derivatives[_derivativeIndex] != address(0))` 

## Non-critical Issues

The parameter `_amount` is used as an input, but not used in the `ethPerDerivative` function, making `_amount` redundant. This is seen in 2 places: [SfrxEth](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L111) and  [WstEth](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L86)