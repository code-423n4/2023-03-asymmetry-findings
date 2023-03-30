# Abstract

Assuming that the number of derivatives will always be lower than 256 (confirmed on Discord by @Toshi), you can save some gas by using `uint8` instead of `uint256` for `derivativeCount` in `SafEthStorage.sol`

# Patch
In file `contracts/SafEth/SafEthStorage.sol`,
from:
`18:     uint256 public derivativeCount; // amount of derivatives added to contract`

to:
`18:     uint8 public derivativeCount; // amount of derivatives added to contract`

You can find a git patch here, against "main" branch: https://pastebin.com/AiqnbfNZ with password "GqaLNmrB82"

Gas saved is then proportional to the number of derivatives set.

# Tests

Results by running the tests.

Before:
stake (min: 437702, max 660515, avg 527253)
unstake (min: 418044, max 571044, avg 516303)

After:
stake (min: 436099, max 658912, avg 525650)
unstake (min: 416518, max 569256, avg 514514)