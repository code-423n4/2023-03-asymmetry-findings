| | issue |
| ----------- | ----------- |
| 1 | [state variables should be cached in stack variables rather than re-reading them from storage](#1-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage) |
| 2 | [can make the variable outside the loop to save gas](#2-can-make-the-variable-outside-the-loop-to-save-gas) |
| 3 | [Avoid a SLOAD optimistically](#3-avoid-a-sload-optimistically) |
| 4 | [using `bool` for storage incurs overhead](#4-using-bool-for-storage-incurs-overhead) |
| 5 | [ Ternary over if ... else](#5-ternary-over-if--else) |
| 6 | [public functions not called by the contract should be declared external instead](#6-public-functions-not-called-by-the-contract-should-be-declared-external-instead) |
| 7 | [should use arguments instead of state variable](#7-should-use-arguments-instead-of-state-variable) |
| 8 | [Functions guaranteed to revert when called by normal users can be marked `payable`](#8-functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable) |
| 9 | [Optimize names to save gas](#9-optimize-names-to-save-gas) |
| 10 | [Non-strict inequalities are cheaper than strict ones](#10-non-strict-inequalities-are-cheaper-than-strict-ones) |
| 11 | [part of the code can be pre calculated](#11-part-of-the-code-can-be-pre-calculated) |


## 1. state variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

`derivatives[i].balance()` is being read twice for each iteration. cache it at the start of the for loop to reduce a complex SLOAD per iteration.
- [SafEth.sol#L73-L74](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L73-L74)

`derivativeCount` is being read twice in #L71 and #L84 so it can be cached before #L71 to reduce one SLOAD
- [SafEth.sol#L71-L84](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71-L84)

same for `derivativeCount` here in #L140 and #L147
- [SafEth.sol#L140-L147](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140-L147)

`totalWeight` is being read every time in the loop but it is not changed in the loop so it is always the same while the loop is going. cache it before the loop to stop SLOADs on every iteration and only do one before the loop.
- [SafEth.sol#L88](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L88)

same for `totalWeight`here in #L150
- [SafEth.sol#L150](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L150)

cache `derivatives[i].balance()` here because the if statement is usually true and inside it will be another complex SLOAD for it. even if the if statement be false we only risk losing 3 gas because of caching it and it is totally worth it. cache `derivatives[i].balance()` before the if statement. this will save lots of gas because it saves gas for every iteration of for loop
- [SafEth.sol#L141-L142](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L141-L142)

`weights[i]` can be cached to possibly reduce a complex SLOAD. it is read in both #L148 and #L149 and usually #L149 will happen so it is worth to risk losing 3 gas for caching to save huge amount of gas. saves gas for every iteration of for loop 
- [SafEth.sol#L148-L149](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L148-L149)

`derivativeCount` is being read 4 times and it should be cached at the start of the function to reduce 3 extra SLOADs. with caching it we can stop 2 reads in #L186 and #L187 but for #L191 and #194 we can use `derivativeCount + 1` or we can update the cached version as well as the statevar near #L188
- [SafEth.sol#L186-L194](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L186-L194)


## 2. can make the variable outside the loop to save gas

make the variable outside the loop and only give the value to variable inside

`weight`
- [SafEth.sol#L85](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L85)

`ethAmount`
- [SafEth.sol#L88](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L88)
- [SafEth.sol#L149](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L149)

`depositAmount`
- [SafEth.sol#L91](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L91)

`derivativeReceivedEthValue`
- [SafEth.sol#L92](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L92)

`derivativeAmount`
- [SafEth.sol#L115](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L115)


## 3. Avoid a SLOAD optimistically

There is a chance that the first part will be true so the second part doesn’t need to be checked, so it is better to use the part that we have first instead of the part that needs to be called.

`if (weights[i] == 0 || ethAmountToRebalance == 0)` should be `if (ethAmountToRebalance == 0 || weights[i] == 0)` for a possible reduce of a complex SLOAD
- [SafEth.sol#L148](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L148)


## 4. using `bool` for storage incurs overhead

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled. Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past

`pauseStaking` and `pauseUnstaking` are inherited and both bool variables consider changing them because it has impact on current contracts in scope but the contract they are declared in is out of scope
- [SafEthStorage.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEthStorage.sol)


## 5. Ternary over if ... else

Using ternary operator instead of the if else statement saves gas.

- [Reth.sol#L212-L215](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L212-L215)

- [SafEth.sol#L79-L81](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L79-L81)


## 6. public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

`name()`
- [WstEth.sol#L41](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L41)
- [SfrxEth.sol#L44](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L44)
- [Reth.sol#L50](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L50)

`ethPerDerivative()`
- [WstEth.sol#L86](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86)
- [Reth.sol#L211](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L211)

`balance()`
- [WstEth.sol#L93](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L93)
- [SfrxEth.sol#L122](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L122)
- [Reth.sol#L221](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L221)


## 7. should use arguments instead of state variable

This will save 100 gas because it gets rid of a storage read and instead uses a argument that we already have which gives the same result

use `_minAmount` instead
- [SafEth.sol#L216](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L216)

use `_maxAmount` instead
- [SafEth.sol#L225](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L225)

use `_pause` instead
- [SafEth.sol#L234](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L234)

use `_pause` instead
- [SafEth.sol#L243](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L243)


## 8. Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

- [WstEth.sol#L48](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48)
- [WstEth.sol#L56](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56)
- [WstEth.sol#L73](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L73)

- [SfrxEth.sol#L51](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51)
- [SfrxEth.sol#L60](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60)
- [SfrxEth.sol#L94](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L94)

- [Reth.sol#L58](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58)
- [Reth.sol#L107](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107)
- [Reth.sol#L156](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L156)

- [SafEth.sol#L138](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138)
- [SafEth.sol#L168](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L168)
- [SafEth.sol#L185](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L185)
- [SafEth.sol#L205](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L205)
- [SafEth.sol#L214](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214)
- [SafEth.sol#L223](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223)
- [SafEth.sol#L232](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232)
- [SafEth.sol#L241](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241)


## 9. Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).
you can use this [tool](https://emn178.github.io/solidity-optimize-name/) to get the optimized version for function and properties signitures 


## 10. Non-strict inequalities are cheaper than strict ones

In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).
consider replacing >= with the strict counterpart > and add `- 1` to the second side

- [Reth.sol#L147](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L147)
- [Reth.sol#L149](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L149)

- [SafEth.sol#L65](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L65)
- [SafEth.sol#L66](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L66)


## 11. part of the code can be pre calculated

these parts of code will use operations to calculate every time they are used, so instead just pre calculate them and use them as constants made in the contract

`1 * 10 ** 16` and `10 ** 18` and `96 * 2` and `5 * 10 ** 17` and `200 * 10 ** 18`

- [WstEth.sol#L35](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L35)
- [WstEth.sol#L60](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L60)
- [WstEth.sol#L87](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L87)

- [SfrxEth.sol#L38](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L38)
- [SfrxEth.sol#L74-L75](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75)
- [SfrxEth.sol#L113](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L113)
- [SfrxEth.sol#L115](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L115)

- [Reth.sol#L44](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L44)
- [Reth.sol#L171](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L171)
- [Reth.sol#L173-L174](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173-L174)
- [Reth.sol#L214-L215](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L214-L215)
- [Reth.sol#L241](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L241)

- [SafEth.sol#L54-L55](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L54-L55)
- [SafEth.sol#L75](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L75)
- [SafEth.sol#L80-L81](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L80-L81)
- [SafEth.sol#L94](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L94)
- [SafEth.sol#L98](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L98)
