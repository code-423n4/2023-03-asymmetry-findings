Report 1:

In the  initialize function described in the contracts Reth.sol, SfrxEth.sol, WstEth.sol and SafEth.sol you are using the _transferOwnership() function to transfer the ownership
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L48
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L42
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L36
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L33

but instead of _transferOwnership(), using the transferOwnership() from the OwnableUpgradeable.sol would decrease the amount of gas needed when calling the function.

----------------------------------------------------------------------------------------------------
Report 2: 

unchecked{derivativeCount++} should be implemented in: 
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L188

the loop can execute faster since the check for overflow is eliminated. This can result in a lower gas cost.

Note: This was not mentionned in https://gist.github.com/muratkurtulus/c7a89b0ef411b5b96dd8af23ccd95dc4
----------------------------------------------------------------------------------------------------
