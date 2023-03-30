## USERS' ETH STUCK IN THE CONTRACT DUE TO UNINITIALIZED VARIABLES
In SatEth.sol, `initialize()` does not have `derivatives` and `weights` setup. Because `stake()` is unpaused upon contract deployment, users could start staking with `derivativeCount` still equal to zero. As a result, all for loops are skipped while `preDepositPrice` is set to 10 ** 18.

In the end, `mintAmount` is assigned 0 while the ETH sent in belongs to the contract that will be irretrievable by anyone.

Suggested fix:

It is recommended initializing `derivatives` and `weights` in [`initialize()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L48-L56).   

## IRRETRIEVABLE ETH IN SAFETH CONTRACT
In conjunction to above finding, users` ETH stuck in SafEth.sol is irretrievable. Neither can the contract owner do anything about it. Additionally, there might be other reasons where ETH would be stuck in the contract, e.g. mistakenly sent into the contract or recipient contract non-existence leading to successful call() but zero ETH sent.   

Suggested fix:

It is recommended having a withdraw function implemented or have [`rebalanceToWeights()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138) refactored such that `address(this).balance` instead of `ethAmountToRebalance = ethAmountAfter - ethAmountBefore` is used when repositing to the derivatives. At least, the unreachable ETH could be distributed to all existing stakers in the system.     

## MODERN MODULARITY ON IMPORT USAGES
For cleaner Solidity code in conjunction with the rule of modularity and modular programming, it is recommended using named imports with curly braces (limiting to the needed instances if possible) instead of adopting the global import approach.

Suggested fix for a contract instance as an example:

[SfrxEth.sol#L4-L9](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L4-L9)

```
import {IDerivative} frpm "../../interfaces/IDerivative.sol";
import {IsFrxEth} from "../../interfaces/frax/IsFrxEth.sol";
import {OwnableUpgradeable} from "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import {IFrxEthEthPool} from "../../interfaces/curve/IFrxEthEthPool.sol";
import {IFrxETHMinter} from "../../interfaces/frax/IFrxETHMinter.sol";
```
## MISSING SLIPPAGE PROTECTION
In SafEth.sol, a slippage option should be provided when users are calling [`stake()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63-L101). This is because [`derivatives[i].ethPerDerivative(derivatives[i].balance())`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L73) is not returning a fixed value but rather the price of derivative in terms of ETH that can be changing albeit less volatile compared to other crypto pair. Nonetheless, this could affect the amount of [`safETH` minted](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L99).

For instance, if the price returned is higher than the one previously called by another staker, `mintAmount` is going to be proportionately lower in the reverse manner. The same rule shall apply the opposoite way if the price returned is lower.

Suggetsed fix:

It is recommended implementing slippage protection in `stake()` so that users could be assured of the minimum amount of safETH they are going to receive.    

## USE MORE RECENT VERSIONS OF SOLIDITY
Lower versions like 0.8.13 are being used in the contracts. For better security, it is best practice to use the latest Solidity version, 0.8.19.

Please visit the versions security fix list in the link below for detailed info:

https://github.com/ethereum/solidity/blob/develop/Changelog.md

## NATSPEC SHOULD BE ADEQUATE
As denoted in the link below:

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). As clearly stated in the Solidity official documentation, in complex projects such as deFi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

## CONTRACT LAYOUT ON FUNCTION WRITINGS COMPLIANCE WITH SOLIDITY'S STYLE GUIDE
As denoted in Solidity's Style Guide:

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

In order to help readers identify which functions they can call, and find the constructor and fallback definitions more easily, functions should be grouped according to their visibility and ordered in the following manner:

constructor, receive function (if exists), fallback function (if exists), external, public, internal, private

And, within a grouping, place the view and pure functions last.

Additionally, inside each contract, library or interface, use the following order:

type declarations, state variables, events, modifiers, functions

Where possible, consider adhering to the above guidelines for all contract instances entailed.

## COMPILER VERSION PRAGMA SPECIFICITY
Non-library contracts and interfaces should avoid using floating pragmas ^0.8.0. Doing this may be a security risk for the actual application implementation itself. For instance, a known vulnerable compiler version may accidentally be selected or a security tool might fallback to an older compiler version leading to checking a different EVM compilation that is ultimately deployed on the blockchain.

## STORAGE GAP FOR UPGRADEABLE CONTRACTS
Consider adding a storage gap at the end of each upgradeable contract. In the event some contracts needed to inherit from them, there would not be an issue shifting down of storage in the inheritance chain. Generally, storage gaps are a novel way of reserving storage slots in a base contract, allowing future versions of that contract to use up those slots without affecting the storage layout of child contracts. If not, the variable in the child contract might be overridden whenever new variables are added to it. This storage collision could have unintended and vulnerable consequences to the child contracts.

This concerns all 2 contracts in scope.

Suggested fix:

It is recommended adding the following code block at the end of the upgradeable contract:

```
    /**
     * @dev This empty reserved space is put in place to allow future versions to add new
     * variables without shifting down storage in the inheritance chain.
     * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
     */
    uint256[49] private __gap;
```
## RENOUNCEABLE OWNERSHIP
When inheriting from Openzeppelin’s OwnableUpgradeable.sol, `renounceOwnership()` is one of the callable functions included. This could pose a risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby denying access to any functionality that is only callable by the owner.

This specifically concerns SatEth.sol.

## TIMELOCK FOR CRITICAL PARAMETER CHANGE
It is a good practice giving time to users to react and adjust to critical changes with a mandatory time window between the changes. The first step is simply broadcasting to users with a specific change that is coming whilst the second step commits that change after an appropriate period of waiting. This would allow time for users opposing to the change to withdraw within the set time frame. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of the owner making a malicious act). Specifically, privileged roles could use front running to make malicious changes just ahead of incoming transactions, or purely accidental negative effects could occur due to the unfortunate timing of changes. 

Here are the instances found.

[SafEth.sol#L138-L195](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138-L195)

## SANITY CHECKS
Zero address and zero value checks should be implemented whenever possible to avoid human errors leading to non-functional calls associated with the mistakes. 

For instance, derivatives added with erroneous address included in [`IDerivative`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L186) instant is going to permanently stay in the system because of the non-deleteable mappings associated with `derivativeCount`. The mistakes made will have these non-functional mappings always looped through and skipped via `continue` in `stake()` and `unstake()`.  

## SOLIDITY COMPILER OPTIMIZATIONS COULD BE PROBLEMATIC
```
hardhat.config.js:
  29  module.exports = {
  30:   solidity: {
  31:     compilers: [
  32:       {
  33:         version: "0.8.13",
  34:         settings: {
  35:           optimizer: {
  36:               enabled: true,
  37:               runs: 1000000
  38
            }
```
Description: Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

Exploit Scenario A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

Recommendation: Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug. Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## SYSTEM DOCUMENTATION AND TECHNICAL SPECIFICATION
Having an up-to-date system design specification and documentation is as important as the system implementation because users and developers heavily rely on them to understand how the entire flow works. For the contest, users could only resort to the code base itself, something the vast majority of users cannot understand. Security assessments depend on a complete technical specification to understand the specifics of how a system works. When a behavior is not specified (or is specified incorrectly), security assessments must base their knowledge in assumptions, leading to less effective review. Maintaining and updating code relies on supporting documentation to know why the system is implemented in a specific way. If code maintainers cannot reference documentation, they must rely on memory or assistance to make high-quality changes. Currently, the only documentation available is a single README file, as well as code comments.

## PAUSABLE CHECKS COULD BE GROUPED INTO A MODIFIER
The [`pauseStaking`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L64) and [`pauseUnstaking`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L109) checks are similar in logic and may be grouped into a modifier for better code structure. In line of this, [`setPauseStaking()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232-L235) and [`setPauseUnstaking()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241-L244) may also be simplified into one setPause function.

## BOOLEAN EXPRESSION AND LITERAL COMPARE
In conjunction to the above suggestion, it is inexpedient comparing a boolean expression to a boolean literal in a require check. Simply use negation, `!`, if it needs to be false. 

Suggested fix:

[SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)

```
64:       require(!pauseStaking, "staking is paused");

109:       require(!pauseUnstaking, "unstaking is paused");
```
  