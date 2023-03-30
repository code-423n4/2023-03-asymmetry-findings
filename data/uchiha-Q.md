|   LOW    | Title                             |
| -------- | --------------------------------- |
| [LOW-01] | Zero input-value check            |
| [LOW-02] | Loss of precision due to rounding |

  
## [L-01] Zero input-value check
## Details
Zero input-value check should be added for the functions stake() and unstake() on SafEth.sol
It should revert if the value is 0.
## Code snippets
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L63
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L108
## Recommendations
function stake() { require(msg.value != 0, "Non-zero value needed");
function unstake(uint256 _safEthAmount) { require(_safEthAmount != 0, "Non-zero value needed");
  
  
## [L-02] Loss of precision due to rounding
## Details
Loss of precision due to rounding
uint256 derivativeAmount = (derivatives[i].balance() * _safEthAmount) / safEthTotalSupply;
uint256 ethAmount = (ethAmountToRebalance * weights[i]) / totalWeight;
## Code snippets
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L115-L116
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L149-L150