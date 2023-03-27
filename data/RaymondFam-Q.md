## Timelock for `adjustWeight()` and `addDerivative()`
Changes made via `adjustWeight()` and `addDerivative()` are sensitive transactions that may go against users` original portfolio plan with Asymmetry Finance. As such, users of the system should have assurances about the behavior of the changed action(s) they’re about to take.

Here are two typical secnarios that could transpire:
- A user decides to stake 100 ETH after seeing a placing of weights to his desire the system has on the the three derivatives, i.e. Reth, SfrxEth and WstEth. His call happens to be inadvertently front run by [`adjustWeight()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175) or [`addDerivative()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-L195) with his ETH now diversified to a different portfolio of staked derivatives.
- The same situation could also happen later, if not front run.   

Consider implementing a time lock by making the weight and derivative parameter changes require two steps with a mandatory time window between them. The first step merely broadcasts to users that a particular change is coming, and the second step commits that change after a suitable waiting period. This allows users that do not accept the change to withdraw their positions in time.

## Modularity on import usages
For cleaner Solidity code in conjunction with the rule of modularity and modular programming, use named imports with curly braces instead of adopting the global import approach.

For example, the import instance below could be refactored conforming to the suggested standards as follows:

[File: WstEth.sol#L4-L8](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L4-L8)

```diff
- import "../../interfaces/IDerivative.sol";
+ import {IDerivative} from "../../interfaces/IDerivative.sol";
- import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
+ import {OwnableUpgradeable} from "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
- import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
+ import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
- import "../../interfaces/curve/IStEthEthPool.sol";
+ import {IStEthEthPool} from "../../interfaces/curve/IStEthEthPool.sol";
- import "../../interfaces/lido/IWStETH.sol";
+ import {IWStETH} from "../../interfaces/lido/IWStETH.sol";
```
## Parameterized address variables
Addresses should be parameterized and initialized instead of being declared as constants.

Here are the contract instances entailed:

[File: Reth.sol#L20-L27](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L20-L27)
[File: SfrxEth.sol#L15-L21](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L15-L21)
[File: WstEth.sol#L13-L18](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L13-L18)

## Unspecific compiler version pragma
For some source-units the compiler version pragma is very unspecific, i.e. ^0.8.13. While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up actually checking a different EVM compilation that is ultimately deployed on the blockchain.

Avoid floating pragmas where possible. It is highly recommend pinning a concrete compiler version (latest without security issues) in at least the top-level “deployed” contracts to make it unambiguous which compiler version is being used. Rule of thumb: a flattened source-unit should have at least one non-floating concrete solidity compiler version pragma.

## Use a more recent version of solidity
The protocol adopts version 0.8.13 on writing contracts. For better security, it is best practice to use the latest Solidity version, 0.8.17.

Security fix list in the versions can be found in the link below:

https://github.com/ethereum/solidity/blob/develop/Changelog.md

## Devoid of system documentation and complete technical specification
A system’s design specification and supporting documentation should be almost as important as the system’s implementation itself. Users rely on high-level documentation to understand the big picture of how a system works. Without spending time and effort to create palatable documentation, a user’s only resource is the code itself, something the vast majority of users cannot understand. Security assessments depend on a complete technical specification to understand the specifics of how a system works. When a behavior is not specified (or is specified incorrectly), security assessments must base their knowledge in assumptions, leading to less effective review. Maintaining and updating code relies on supporting documentation to know why the system is implemented in a specific way. If code maintainers cannot reference documentation, they must rely on memory or assistance to make high-quality changes. Currently, the only documentation available is a single README file, as well as code comments.

## Empty event
The following event has no parameter in it to emit anything. Although the standard global variables like block.number and block.timestamp will implicitly be tagged along, consider adding some relevant parameters to the make the best out of this emitted event for better support of off-chain logging API.

Here is an instance entailed:

[File: SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)

```solidity
34:    event Rebalanced();

154:        emit Rebalanced();
```
