# Asymmetry Finance contest - Gas Optimizations

# List of Gas Optimizations found


### [G‑01]: ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops

There are few instances in the code where we have used i++ in the for loops. The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop.
Since the code is written with solidity ^0.8.13 version this gas optimization is valid here.

**There are 7 instances of this issue in SafEth.sol**


### [G‑02]: Use custom errors rather than revert()/require() strings to save gas

Using a custom error instance will usually be much cheaper than a string description, because you can use the name of the error to describe it, which is encoded in only four bytes. A longer description can be supplied via NatSpec which does not incur any costs.

Since the code is written with solidity ^0.8.13 version this gas optimization is valid here.

**There are 10 instances of this issue**

### [G‑03] Don’t compare boolean expressions to boolean literals
Eg: 
```javascript
if (<x> == true) => if (<x>)
```
```javascript
 if (<x> == false) => if (!<x>).
```
**There are 2 instances of this issue**
This will save atleast 18 gas
