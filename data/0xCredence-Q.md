1.) Require that `minAmount` and `maxAmount` that can be deposited are not set to zero and also require that max amount is greater than min amount in both setMinAmount and setMaxAmount functions in order to prevent unexpected behaviours.
See https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L215
 and https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L224


2.)Require that `setPauseStaking` and `setPauseUnstaking` values are not equal to previous values to ensure that the correct intended value was added and the value in fact changed.
See https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L233 and https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L242
