# 1. Use newer versions of solidity

Consider using the latest version of solidity as newer versions have bug fixes, as well as new features.

To see what the latest versions have to offer check out the [Change Log](https://github.com/ethereum/solidity/blob/develop/Changelog.md)

Affected lines of code:

- [WstEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L2)

- [SfrxEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L2)

- [SafEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L2)

- [Reth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L2)

# 2. Improve the Readability by following modularity principles

To make your Solidity code more readable, update your import statements to only include the specific contracts or objects that you need.

This practice, known as modular programming, helps to keep the code cleaner, maintainable and more organized.

An example of this syntax is shown below:

```solidity
import {contract1, contract2} from "filename.sol";
```

Affected line of code:

- [WstEth.sol#L4](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L4-L8)

- [Reth.sol#L4-L15](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L4-L15)
- [SafEth.sol#L4-L11](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L4-L11)
- [SfrxEth.sol#L4-L9](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L4-L9)

# 3. Unspecific Compiler Version Pragma

It is generally not recommended to use floating pragmas (i.e. pragmas that do not specify a specific compiler version) in contracts that are not intended to be used as libraries.

This is because using floating pragmas in application contracts can pose a security risk.

For example, a known vulnerable compiler version may be selected by mistake, or security tools might revert to an older compiler version that produces a different EVM compilation than the one intended to be deployed on the blockchain.

To avoid these potential issues, consider specifying a specific compiler version in your pragmas.

So instead of using a floating pragma like `pragma solidity ^0.8.0;`, it is better to use a concrete compiler version like `pragma solidity 0.8.17;`.

More information can be found in the following links:

- [Consensys Audit of 1inch](https://consensys.net/diligence/audits/2020/12/1inch-liquidity-protocol/#unspecific-compiler-version-pragma)
- [Solidity docs](https://docs.soliditylang.org/en/latest/layout-of-source-files.html#version-pragma)
- [Solidity Specific](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

Affected lines of code:

- [WstEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L2)

- [SfrxEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L2)

- [SafEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L2)

- [Reth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L2)

# 4. Follow the function order of the solidity style guide

The [Solidity style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) recommends the following function order:

- constructor

- receive function (if exists)

- fallback function (if exists)

- external

- public

- internal

- private

Within a grouping, place the `view` and `pure` functions last.

This is because "Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier." -solidity style guide

Affected files:

- [WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol)
- [SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol)
- [SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)
- [Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol)
