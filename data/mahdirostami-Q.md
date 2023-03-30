## Non-library files should use fixed compiler versions

Contracts should be compiled using a fixed compiler version. Locking the 
pragma helps ensure that contracts do not accidentally get deployed using a 
different compiler version with which they have been tested the most.

4 instances
affected code:
```
pragma solidity ^0.8.13;
contracts/SafEth/derivatives/WstEth.sol
contracts/SafEth/derivatives/SfrxEth.sol
contracts/SafEth/SafEth.sol 
contracts/SafEth/derivatives/Reth.sol 
```

MITIGATION
Used a fixed compiler version.

## Function order
Functions should be ordered following the Solidity conventions: 
receive() function should be placed after the constructor and before every other function.

4 instances
affected code:
```
contracts/SafEth/derivatives/WstEth.sol
contracts/SafEth/derivatives/SfrxEth.sol
contracts/SafEth/SafEth.sol 
contracts/SafEth/derivatives/Reth.sol 
```

MITIGATION
Place the receive() and fallback() functions after the constructor, before all the other functions.

## Receive function

These contracts have a receive() function, but do not have any 
withdrawal function. Any Manifest mistakenly sent to this contract would be locked.

4 instances
affected code:
```
contracts/SafEth/derivatives/WstEth.sol
contracts/SafEth/derivatives/SfrxEth.sol
contracts/SafEth/SafEth.sol 
contracts/SafEth/derivatives/Reth.sol 
```
MITIGATION
Add require(0 == msg.value) in receive() or remove the function altogether.

## Use named imports instead of plain import “file.sol”

4 instances
affected code:
```
contracts/SafEth/derivatives/WstEth.sol
contracts/SafEth/derivatives/SfrxEth.sol
contracts/SafEth/SafEth.sol 
contracts/SafEth/derivatives/Reth.sol 
```
## Prevent division by 0
On several locations in the code precautions are not being taken for not dividing by 0, this will revert the code.
There is no check on totalWeight. 

2 instances
affected code:
```
uint256 ethAmount = (msg.value * weight) / totalWeight;
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L88

uint256 ethAmount = (ethAmountToRebalance * weights[i]) / totalWeight;
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L149-L150
```

MITIGATION:
1.check totalWeight in adjustWeight and addDerivative functions
```
require(tatalWeight = !0);
```
or
2.check totalWeight in rebalanceToWeights and stake functions



