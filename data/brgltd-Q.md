# [01] Add checks for weight values

Currently it's possible to set any value for the weights. Some combinations for weights could result in issues while calculating `ethAmount`.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L169

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L187

For example, assuming the minimum value for msg.value, three derivatives and strange values for the weights.

```
msg.value = 5e17 = 0.5e18
weight1 = 5e17   = 0.5e18
weight2 = 19e18  =  19e18
weight3 = 19e19  = 190e18

ethAmount = (msg.value * weight) / totalWeight
5e17 * 5e17 / (5e17 + 19e18 + 19e19)
```

This would result in `1193317422434367.5` which would round down in solidity.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L88

## Recommendation

Add checks for min and max values for weights.

# [02] Lack of method to remove derivatives

In case a derivative gets added by mistake or with incorrect parameters, currently this derivative would remain stuck in `SafEth.sol`.

## Recommendation

Consider adding a method that allows removing derivatives from `SafEth.sol`.

# [03] Reentrancy for `SafEth.unstake()`

There is a reentrancy possibility in `SafEth.unstake()` where the tokens are burned only after the derivative withdraw.

If the `derivatives[i].withdraw()` external call where to reenter into `SafEth.unstake()`, the `safEthAmount` is still not updated, since the `_burn()` is only called after, and the function doesn't contain a nonReentranct modifier.

Note: this would only be an issue for a malicious derivative contract.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113-L120

## Recommendation

Call `_burn()` before `derivatives[i].withdraw()` in `SafEth.unstake()`.

# [04] Unbounded loop

There are multiple instances of loops executing external calls where the number of iterations is unbounded and controlled by the number of derivatives. This is not an issue on the current setup, since there are only three derivatives. 

However, if a large amount of derivative gets added, functionalities like `stake()` and `unstake()` could run out of gas and revert.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84-L96

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113-L119

## Recommendation

Limit the maximum number of derivatives that can be added.

# [05] Emit events before external calls

Multiple functions in the project emit an event as the last statement. Wherever possible, consider emitting events before external calls. In case of reentrancy, funds are not at risk (for external call + event ordering), however emitting events after external calls can damage frontends and monitoring tools in case of reentrancy attacks.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63-L101

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108-L129

# [06] Pragma float

All contracts in scope are floating the pragma version.

Locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

Note that pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or a package.

# [07] Lack of address(0) checks

Input addresses should be checked against address(0) to prevent unexpected behavior.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L33-L36

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L42-L45

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L36-L38

# [08] Lack of setter functions for third party integrations

Misdeployed values can cause failure of integrations. One addition that can be made is to add setter functions for the owner to update these addresses if necessary.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L13-L18

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L20-L27

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L14-L21

# [09] Don't allow adding a new derivative when staking/unstaking is paused

When the system is in pause mode, e.g. staking and unstaking is blocked, consider adding a check to prevent new derivatives from being added, e.g.

```
require(!pausedStaking && !pauseUnstaking, "error");
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-L195

# [10] Critical changes should use a two-step pattern and a timelock

Lack of two-step procedure for critical operations leaves them error-prone.

Consider adding a two-steps pattern and a timelock on critical changes to avoid modifying the system state.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223-L226

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232-L235

# [11] Lack of event for parameters changes

Adding an event will facilitate offchain monitoring when changing system parameters.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48-L50

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58-L60

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53

# [12] Lack of old and new value for events related to parameter updates

Events that mark critical parameter changes should contain both the old and the new value.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175

# [13] Check for stale values on setter functions

Add a check ensuring that the new value if different than the current value to avoid emitting unnecessary events.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214-L217

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223-L226

# [14] Calls for retrieving the balance can be cached

`derivatives[i].balance()` can be cached on the following instance.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L141-L142

# [15] Variable being initialized with the default value

Unsigned integers will already be initalized with zero on their declaration, e.g. there's no need to manually assign zero.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L68

# [16] Unnecessary calculation

Multiplying by `10**18` and dividing by `10**18` is not needed on L215 of `Reth.sol`.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215

# [17] Missing unit tests

The derivative contracts don't have all functions and branches covered.

It is crucial to write tests with possibly 100% coverage for smart contracts. It is recommended to write tests for all possible code flows.

# [18] Incorrect NATSPEC

`SafEth.adjustWeight()` contains an incorrect @notice.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L158

# [19] Large or complicated contracts should implement fuzzing tests

Large code bases, or code with complicated math, or complicated interactions between multiple contracts, should implement fuzzing tests.

Fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. 

Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and fuzzers, with properly and extensively-written invariants, can close this testing gap significantly.

# [20] In `SafEth.adjustWeight()` there's no need to loop all derivatives

It's possible to decrease the old weight and increase the new weight to compute `localTotalWeight` and update `totalWeight`.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175

# [21] Variable shadowing

Consider renaming the variable `totalSupply` in `SafEth.stake()`, since it's being shadowed by `ERC20Upgradeable.totalSupply()`.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L77

# [22] Usage of return named variables and explicit values

Some functions return named variables, others return explicit values.

Following function returns an explicit value.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L228-L242

Following function return returns a named variable.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L83-L102

Consider adopting the same approach throughout the codebase to improve the explicitness and readability of the code.

# [23] Imports can be group

Consider grouping the imports, e.g. first libraries, then interfaces, the storage.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L4-L11

# [24] Order of functions

The solidity [documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) recommends the following order for functions:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private

The receive() functions are currently in the bottom on the contract.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L246

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L97

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L244

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L126

# [25] Add a limit for the maximum number of characters per line

The solidity [documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) recommends a maximum of 120 characters.

Consider adding a limit of 120 characters or less to prevent large lines.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L142

# [26] Use scientific notation rather than exponentiation

Scientific notation can be used for better code readability, e.g. consider using using `10e18` and `10e17` instead of `10**18` and `10**17`.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L54-L55

# [27] Specify the warning being disabled by the linter

Consider also adding the name of the warning being disabled, e.g. `// solhint-disable-next-line warning-name`.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L123-L126

# [28] Replace `variable == false` with `!variable`

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L64

# [29] Interchangeable usage of uint and uint256

Consider using only one approach, e.g. only uint256.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L91-L92

# [30] Can use ternary

The following instance can use a ternary expression instead of a conditional.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L78-L81

# [31] Package @balancer-labs/balancer-js is not used

The package @balancer-labs/balancer-js is deprecated and balancer recommends to use the @balancer-labs/sdk package instead. Also, this package is currently unused on the tests and deployment setups. Consider removing this package. 

This is not a vulnerability, but removing this package is beneficial to the project, since unnecessary packages incurs overhead and increases the project download time and size.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/package.json#L78
