# [G-01-1] Use function for getting rocketDepositPoolAddress
In `Reth` contract there are multiple calls to get `rocketDepositPoolAddress`, but there is no function for it. My suggestion is to implement a simple function that returns the address for the Reth deposit pool and use it it these places.
  
[Reth.sol/L121-L127](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L121-L127)
[Reth.sol/L158-L164](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L158-L164)

# [G-01-2] Use function for getting rETH address

There is an inline call for getting `rETH` address in `deposit` while there is already a function for it.
My suggestion is to use the `rethAddress()` function [here](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L187-L196) instead of calling the inline implementation.

    RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(rethAddress());

# [G-02] Checks first, interactions second
In this manner, we will first check if there is any  `weight` and after that we will create the `derivative`.
Finding [here](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L86)
Change places from:

    IDerivative derivative = derivatives[i];
    if (weight == 0) continue;                         
to: 

    if (weight == 0) continue;
    IDerivative derivative = derivatives[i];
       
# [G-03] Use nested if() statements

Using nested saves on deployment costs. Also it makes easier to read code and provides better coverage reports.
Finding [here](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L148)
Recommendations:

    if (weights[i] == 0) continue; 
    if (ethAmountToRebalance) == 0 continue;


# [G-04] Use unchecked when making calculations that won't over/under flow
Using unchecked is a great solution on saving a little bit of gas. It is recommended to use it every time that there is a secure operation that won't overflow/underflow.

[SafEth/SafEth.sol/L95](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L95)
[SafEth/SafEth.sol/L172](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L172)
[SafEth/SafEth.sol/L188](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L188)
[SafEth/SafEth.sol/L192](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L192)

