## RocketPool price manipulation

Function [`poolPrice`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L228) gets the `RETH`'s price by calling for the current pool's price which could be manipulated using flashloan.

An attacker, as well as protocol users, would suffer losses.

This scenario is unlikely, but to reduce any possibilities to zero, I would recommend using TWAP.