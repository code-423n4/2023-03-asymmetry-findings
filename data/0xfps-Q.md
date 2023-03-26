# [L-01] Value multiplied and divided by the same factor.

## Finding.
In the function [here](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L211-L216), the value of the `poolPrice()` is multiplied by `10**18` and again divided by `10**18`, these two `10**18` cancels out.

## Suggestion.
Return `poolPrice()` instead.