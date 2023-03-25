### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] | Usage of uint/int smaller than 32 bytes (256 bits) cost more gas|6 |
|G-02|Using `ternary` operator instead of the `if` `else` statement saves gas|2|




## [G-01]Usage of uints/ints smaller than 32 bytes (256 bits)
State variable declared as uint/int smaller than 32 bytes (256 bits) will cost more gas when used in the code logic as the compiler must first convert them back to uint256 before using them, the only case where using smaller uint/int size matter is when you store many state variables in the same storage slot otherwise it's better to declare them as uint256 to save gas in functions calls.

in `contracts/SafEth/SafEth.sol `
`Example:`
```diff
-       for (uint i = 0; i < derivativeCount; i++) {
+       for (uint256 i = 0; i < derivativeCount; i++) {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L84
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L92
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L147
File:`contracts/SafEth/derivatives/Reth.sol`
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L171
## [G-02]Using ternary operator instead of the if else statement saves gas.


Here is a specific instance entailed:
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L212-L215
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L79-L81




