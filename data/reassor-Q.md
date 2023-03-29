# List
## Low Risk
1. Incorrect private functions naming
2. Missing validation in setMaxSlippage
3. Critical address change
4. Missing validation in setMinAmount and setMaxAmount
5. Changing derivatives number
6. Derivatives contracts are missing events

## Non-Critical Risk
7. Confusing naming poolPrice and canPoolDeposit
8. Missing natspec
9. The contracts use unlocked pragma
10. Use scientific notation for big numbers
11. Usage of magic numbers
12. Short derivatives names
13. Protocol only supports derivatives with 18 decimals


# 1. Incorrect private functions naming
## Risk
Low

## Impact
Contract `Reth` implements multiple private functions that do not follow correct naming convention. This makes code less readable and more prone to errors. Private function names should start with underscore `_`.

## Proof of Concept
`Reth.sol`:
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L66
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L83-L89
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L120
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L228

## Recommended Mitigation Steps
It is recommended to start naming of all private functions with underscore `_`.


# 2. Missing validation in setMaxSlippage
## Risk
Low

## Impact
Function `SafEth.setMaxSlippage` is missing validation of `_slippage` parameter which might be accidentally set to incorrect value.

## Proof of Concept
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202-L208

## Recommended Mitigation Steps
It is recommended to add validation to parameter `_slippage` to ensure that its value is between well defined range.


# 3. Critical address change
## Risk
Low

## Impact
Changing critical addresses such as ownership should be a two-step process where the first transaction (from the old/current address) registers the new address (i.e. grants ownership) and the second transaction (from the new address) replaces the old address with the new one. This gives an opportunity to recover from incorrect addresses mistakenly used in the first step. If not, contract functionality might become inaccessible.

## Proof of Concept
`SafEth.sol`:
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L18

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
It is recommended to implement two-step process for changing ownership.


# 4. Missing validation in setMinAmount and setMaxAmount
## Risk
Low

## Impact
Functions `SafEth.setMinAmount` and `SafEth.setMaxAmount` are missing validation for the `_minAmount` and `_maxAmount` parameters which might be accidentally set to incorrect values.

## Proof of Concept
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L214-L217
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L223-L226

## Recommended Mitigation Steps
It is recommended to validate parameters `_minAmount` and `_maxAmount` to make sure that their values are in well defined ranges.


# 5. Denial of service - for loops over derivatives
## Risk
Low

## Impact
The protocol works on derivatives that mapping index keep increasing on adding new derivative. This might lead to denial of service conditions since protocol in functions such as `stake` or `unstake` iterates over all derivatives, also over these that were removed and their weight was set to `0`.

## Proof of Concept
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113-L119

## Recommended Mitigation Steps
Consider redesigning protocol in a away it will be not iterating over large number of derivatives.


# 6. Derivatives contracts are missing events
## Risk
Low

## Impact
All derivatives contracts are not implementing events for critical functions. Lack of events emission makes it difficult for off-chain applications to monitor the protocol.

## Proof of Concept
`Reth.sol`:
* Slippage change - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L58-L60
* Withdraw - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L107-L114
* Deposit - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L156-L204

`SfrxEth.sol`:
* Slippage change - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53
* Withdraw - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L60-L88
* Deposit - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L94-L106

`WstEth.sol`:
* Slippage change - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48-L50
* Withdraw - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56-L67
* Deposit - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L73-L81

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
It is recommended to add missing events to listed functions.


# 7. Confusing naming poolPrice and canPoolDeposit
## Risk
Non-Critical

## Impact
Contract `Reth` implements functions `poolPrice` and `poolCanDeposit` that have confusing names which makes code less readable and might lead to errors. Function `poolPrice` refers to the price from the uniswap pool, while `poolCanDeposit` checks if its possible to deposit to the Rocket Pool.

## Proof of Concept
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L120
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L228

## Recommended Mitigation Steps
It is recommended to change the name of the functions so they reflect which pools they refer to.


# 8. Missing natspec
## Risk
Non-Critical

## Impact
Multiple contracts are missing natspec comments which makes code more difficult to read and prone to errors.

## Proof of Concept
`SafEth.sol`
* Incorrect natspec - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L157-L164

`Reth.sol`:
* Missing `@return` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L47-L50
* Missing `@return` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L62-L66
* Missing `@return` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L75-L83
* Missing `@param amount` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L104-L107
* Missing `@return` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L116-L120
* Missing `@return` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L152-L156
* Missing `@return` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L206-L211
* Missing `@return` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L218-L221
* Missing `@return` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L225-L228

`SfrxEth.sol`:
* Missing `@return` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L41-L44
* Missing `@param _slippage` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L48-L51
* Missing `@return` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L90-L94
* Missing `@param _amount` and `@return` params - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L108-L111
* Missing `@return` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L119-L122

`WstEth.sol`:
* Missing `@return` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L38-L41
* Missing `@param _slippage` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L45-L48
* Missing `@param _amount` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L52-L56
* Missing `@return` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L69-L73
* Missing `@param _amount` and `@return` params - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L83-L86
* Missing `@return` param - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L90-L93

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
It is recommended to add missing natspec comments.


# 9. The contracts use unlocked pragma
## Risk
Non-Critical

## Impact
As different compiler versions have critical behavior specifics if the contract gets accidentally deployed using another compiler version compared to one they tested with, various types of undesired behavior can be introduced.

## Proof of Concept
* `SafEth.sol` - `^0.8.13` - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L2
* `Reth.sol` - `^0.8.13` - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L2
* `SfrxEth.sol` - `^0.8.13` - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L2
* `WstEth.sol` - `^0.8.13` - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L2

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
Consider locking compiler version, for example `pragma solidity 0.8.17`.


# 10. Use scientific notation for big numbers
## Risk
Non-Critical

## Impact
All contracts are using math calculations to express big numbers instead of using scientific notation.

## Proof of Concept
`SafEth.sol`:
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L54-L55
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L75
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L80-L81
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L94
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L98

`Reth.sol`:
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L44
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L171
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L173-L174
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L214-L215

`SfrxEth.sol`:
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L38
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L113
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L115

`WstEth.sol`:
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L35
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L60
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L87

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
It is recommended to use scientific notation, for example: `1e18`.


# 11. Usage of magic numbers
## Risk
Non-Critical

## Impact
The protocol is using a lot of "magic numbers" which makes code less readable and more prone to errors.

## Proof of Concept
`SafEth.sol`:
* Use `0.5 ether` and `200 ether` - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L54-L55

`Reth.sol`:
* Use `POOL_FEE` constant - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L180
* Use `POOL_FEE` constant - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L238

`SfrxEth.sol`:
* Use constant values `ETH=0` and `FRX=1` for exchange - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L77-L82
* Use constant values `ETH=0` and `STETH=1` for exchange - https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L61

## Recommended Mitigation Steps
Consider adding constant variables with self-explanatory names.


# 12. Short derivatives names
## Risk
Non-Critical

## Impact
The protocol is using short names for derivatives such as "Lido" or "Frax" which might lead to confusion.

## Proof of Concept
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L51
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L45
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L42

## Recommended Mitigation Steps
Consider adding more detailed names for derivatives.


# 13. Protocol supports only derivatives with 18 decimals
## Risk
Non-Critical

## Impact
The protocol supports only derivatives that use 18 decimals. This assumption is carried in all calculation performed by the protocol.

## Proof of Concept
`SafEth.sol`:
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71-L75
* https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L92-L94

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
Consider adding support for derivatives with different number of decimals.
