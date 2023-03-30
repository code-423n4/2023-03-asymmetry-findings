# 1
## Description: use tenary Operator to replace if/else statement.
 The code,
 ` uint256 preDepositPrice; // Price of safETH in regards to ETH
         if (totalSupply == 0)
            preDepositPrice = 10 ** 18; // initializes with a price of 1
         else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;' cost `(gas: 612742)` gas.

While if tenary operator is used it cost `(gas: 612732)`gas.

`preDepositPrice = totalSupply == 0 ? 10 ** 18 : (10 ** 18 * underlyingValue) / totalSupply;`

the gas report snapshot --diff is shown below:
`testgas() (gas: -10 (-0.002%)) `

## context[SafEth.sol#L78-L81](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L78-L81)

## Recommendation: 
use the tenary operator to save more gas in running the stake function.

# 2.

## Description. Looping through a state variable consume more gas than looping through a memory.
## context: 4 instances [L71](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71), [L113](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113),[L147](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L147), [L191](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L191).

## Recommendation.
it is advisable to cache the derivateCount in memory and loop through the memory variable instead.
looping through the meomry variable for the above instances saves gas.
for example: ` uint _derivativeCount = derivativeCount` 
then loop through the memory variable
for (uint i = 0; i < _derivativeCount; i++) {}
here is the gas difference for all the 4 instances if you cache the variable in memory.
    `testGasOptimized() (gas: -696 (-0.060%)) 
     Overall gas change: -696 (-0.060%)`
this report is done using foundry to generate gas difference.