
## Recalculation of `totalWeight`
In [`SafEth.adjustWeight`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L165-L175) and [`SafEth.addDerivative`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182-L195), instead of looping through each weight to sum them to recalculate `totalWeight`, simply remove the old weight and add the new weight.

## `weight == 0` can be checked earlier
Swap [L86 and L87](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L86-L87).

## move `ethAmountToRebalance == 0` outside for-loop
[`ethAmountToRebalance == 0`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L148) only need to be checked once, so move it outside the for-loop