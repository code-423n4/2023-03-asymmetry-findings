## Low Risk Issues Summary
| Number |Issues Details |
|:--:|:-------|
|[L-01]| No storage gap for upgradable contracts might lead to storage slot collision
|[L-02]| Missing checks for address(0x0)
|[L-03]| Missing event for critical parameter change
|[L-04]| Critical address changes should use two-step procedure
|[L-05]| Unbounded loop while iterating with `derivativeCount`
|[L-06]| No limit for `maxSlippage()` functions
 
***

## Non-Critical Issues Summary
| Number |Issues Details |
|:--:|:-------|
|[NC-01]| Use latest Solidity version
|[NC-02]| Use stable pragma statement
|[NC-03]| Use named imports instead of plain `import FILE.SOL`
|[NC-04]| NatSpec is incomplete
|[NC-05]| Update external dependency to latest version
|[NC-06]| Solidity compiler optimizations can be problematic
|[NC-07]| Initial value check is missing in Set Functions
|[NC-08]| You can use named parameters in mapping types

***

## [L-01] No storage gap for upgradeable contracts might lead to storage slot collision

## Description

For upgradeable contracts, there must be storage gap to “allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments”. 
Otherwise, it may be very difficult to write new implementation code. 
The storage gap is essential for upgradeable contract because “It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments”. 

Without storage gap, the variable in the contract contract might be overwritten by the upgraded contract if new variables are added.  This could have unintended and very serious consequences to the child contracts.

## Recommendations

Consider defining an appropriate storage gap in each upgradeable parent contract at the end of all the storage variable definitions as follows:

```solidity
	uint256[50] __gap; // gap to reserve storage in the contract for future variable additions
```
***

## [L-02] MISSING CHECKS FOR ADDRESS(0X0)

Lack of zero-address validation on address parameters may lead to transaction reverts, waste gas, require resubmission of transactions and may even force contract redeployments in certain cases within the protocol.

```solidity
contracts/SafEth/derivatives/WstEth.sol
33: function initialize(address _owner) external initializer {

contracts/SafEth/derivatives/SfrxEth.sol
36: function initialize(address _owner) external initializer {
37:     _transferOwnership(_owner);
182: function addDerivative(
183:     address _contractAddress,

contracts/SafEth/derivatives/Reth.sol
42: function initialize(address _owner) external initializer {
43:     _transferOwnership(_owner);
```

#### Recommended Mitigation Steps

Consider adding explicit zero-address validation on input parameters of address type.
***

## [L-03] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

```solidity
contracts/SafEth/derivatives/WstEth.sol

48: function setMaxSlippage(uint256 _slippage) external onlyOwner {
49:     maxSlippage = _slippage;
50: }


contracts/SafEth/derivatives/SfrxEth.sol
51: function setMaxSlippage(uint256 _slippage) external onlyOwner {
52:     maxSlippage = _slippage;
53: }

contracts/SafEth/derivatives/Reth.sol
58: function setMaxSlippage(uint256 _slippage) external onlyOwner {
59:     maxSlippage = _slippage;
60: }
```
***

## [L-04] CRITICAL ADDRESS CHANGES SHOULD USE TWO-STEP PROCEDURE

The critical procedures should be a two-step process.
```solidity 
contracts/SafEth/derivatives/WstEth.sol
33: function initialize(address _owner) external initializer {
34:     _transferOwnership(_owner);

contracts/SafEth/derivatives/SfrxEth.sol
36: function initialize(address _owner) external initializer {
37:     _transferOwnership(_owner);

contracts/SafEth/derivatives/Reth.sol
42: function initialize(address _owner) external initializer {
43:     _transferOwnership(_owner);
```

**Recommended Mitigation Steps**
Lack of two-step procedure for critical operations leaves them error-prone. Consider adding a two- step procedure on the critical functions.
***
## [L-05] Unbounded loop while iterating with `derivativeCount`

In [SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol) we have few function which iterating the state variable `derivativeCount` and it is possible unbounded loop.
New `derivativeCount` are added in `addDerivative()` function, without a maximum size limit.

You can add a maximum number of `derivativeCount` that can be added, to prevent unbounded loop.
***
## [L-06] No limit for `maxSlippage()` functions

In `WstEth.sol`, `SfrxEth.sol`, `Reth.sol` and `SafEth.sol` functions. 
These functions allows the owner of the smart contract to set the maximum slippage for the contract without any checks for a maximum value. This means that it is possible for the owner to set an unreasonably high value for `maxSlippage`, which could potentially result in users losing a significant amount of funds when they execute transactions on the smart contract.

**Recommended Mitigation Steps**
It would be a good security practice to add a check for a maximum value to ensure that the owner cannot set an unreasonably high value for `maxSlippage`. This would help to protect users and prevent potential losses due to large slippage values.
***


## [NC-01] Use latest Solidity version

Solidity pragma versioning should be upgraded to latest available version. 
***

## [NC-02] Use stable pragma statement

Using a floating pragma statement `^0.8.13` is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode.
***

## [NC-03] USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT FILE.SOL`

**Recommendation:**
`import {contract1 , contract2} from "filename.sol";
***

## [NC‑04] NatSpec is incomplete

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
You also need to add return parameters in NatSpec comments.
***

## [NC-05] Update external dependency to latest version

The latest version is 4.8.2

package.json:
```solidity
82: "@openzeppelin/contracts": "^4.8.0",
83: "@openzeppelin/contracts-upgradeable": "^4.8.1",
```
***

## [NC-06] Solidity compiler optimizations can be problematic

```solidity
hardhat.config.ts

27:    optimizer: {
28:        enabled: true,
29:        runs: 100000,
30:      },
```
 
Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.
***

## [NC-07] Initial value check is missing in Set Functions

```solidity
contracts/SafEth/derivatives/WstEth.sol
48: function setMaxSlippage(uint256 _slippage) external onlyOwner {

contracts/SafEth/SafEth.sol
202: function setMaxSlippage(
203:     uint _derivativeIndex,
204:     uint _slippage

214: function setMinAmount(uint256 _minAmount) external onlyOwner {
215:     minAmount = _minAmount;
```
Checking whether the current value and the new value are the same should be added.
***
## [NC-8] You can use named parameters in mapping types

From Solidity [0.8.18](https://blog.soliditylang.org/2023/02/01/solidity-0.8.18-release-announcement/) you can use named parameters in mapping types. This will make the code much more readable.