## No address 0 check in  function addDerivative

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182-L195

## No restriction on the weight of a derivative

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L165-L175



## Precision loss in ethamount if the weights are denominated too high (e.g 10**18)

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L88

If totalweights is too high (e.g. 1000 * (10**18)), then  ethamount will revert in some situations

Recommendation: Make sure that weight are not denominated too high


## Users do not have time to react if rebalance weights is called

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L138-L155 

Recommendation: Give users lag time to withdraw if they do not agree with the weight rebalance





