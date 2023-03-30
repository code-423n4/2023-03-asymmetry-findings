# Gas Optimization Report

## ++i costs less gas compared to i++ 
++i costs less gas compared to i++ or i += 1 (about 5 gas per iteration). This statement is true even with the optimizer enabled.

you can change in the lines below:

* Lines 71, 84, 113, 140, 147, 171, 191 of contract safETH.sol


## Update de compilar to 8.4
new versions of Solidity often come with gas optimizations to improve the efficiency, speed of smart contract execution on the Ethereum network. with that been said let go to other gas optimization.

## The increment in for loop post condition can be made unchecked
In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

the for loop post condition, i.e., i++ involves checked arithmetic, which is not required. This is because the value of i is always strictly less than length <= 2**256 - 1. Therefore, the theoretical maximum value of i to enter the for-loop body is 2**256 - 2. This means that the i++ in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks. You should manually add 

for (uint i = 0; i < length;) {
    // do something that doesn't change the value of i
    unchecked { i++; }
}

that can  be added next to the following lines:

- Lines 71, 84, 113, 140, 147, 171, 191 of contract safETH.sol




