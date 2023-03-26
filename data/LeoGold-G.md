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