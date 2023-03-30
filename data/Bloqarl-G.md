## Gas Optimization report

# [G-01] Avoid comparing a bool to true/false
It is not necessary to compare `pauseUnstaking` and `pausestaking` to true or false in the require(). Since they are booleans it is enough to write `require(!pauseUnstaking)`. 

There are two instances in SafEth.sol. 

Gas saved per instance: 18 gas

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L64
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L109


# [G-02] <x> += <y> Costs More Gas Than <x> = <x> + <y>
Inside `stage()` function that it's going to be used very often we can save some gas by using `x = x + y` instead of `x += y`.

There are four instances in `SafEth.sol` and two of those instances are inside a for loop. Which means that it can be saved an important amount of gas.
```
for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
```

Gas saved per instance: 13 gas

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L72
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L95
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L172
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L192


# [G-03] Not using the named return variables in a function returns, wastes gas
In the `deposit()` function it is returned a `uint256`. If we use a named parameter instead of using the return keyword, some extra gas can be saved. Considering that it is a function that will be used a lot, it will be important change.

There are three instances. In the `deposit()` method of `Reth.sol`, `SfrxEth.sol` and `WstEth.sol`

Gas saved per instance: 13 gas

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L156
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L94
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L73


# [G-04] No need to multiply and divide `poolPrice()` with 10**18 in the same operation
In the code we have this `return (poolPrice() * 10 ** 18) / (10 ** 18);` where the result is going to be always `poolPrice()` as the `10**18` will cancel each other.

There is one instance in `Reth.sol` 

Gas saved per instance: 391 gas

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L215

# [G-05] No need to assign variable to zero, as it is by default
In a few methods from `SafEth.sol` it has been declared a uint256 and at the same time assigned zero (for instance `uint256 underlyingValue = 0;`). This is not necessary and wastes gas because the uint256 by default when declared is 0.

There are four instances in `SafEth.sol`

Gas saved per instance: 8 gas

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L68
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L83
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L170
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L190

