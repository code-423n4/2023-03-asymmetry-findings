# GAS OPTIMIZATION REPORT
---
### Summary of optimizations
| Number | Issue details | Instances |
|---|---|:---:|
| [G-1](#G1) |Usage of  `uint`s/`int`s smaller than 32 bytes (256 bits) incurs overhead. | 1
| [G-2](#G2) |Multiple accesses of a mapping/array should use a local variable cache. | 7
| [G-3](#G3) |`>=` costs less gas than `>`. | 2
| [G-4](#G4) |`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables | 4
| [G-5](#G5) |Using fixed bytes is cheaper than using  `string`. | 5

*Total: 5 issues.*

---
### Gas Optimizations
### <a id=G1>[G-1]</a> Usage of  `uint`s/`int`s smaller than 32 bytes (256 bits) incurs overhead.

##### Description
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

##### Recommendation
Use `uint256` instead.

##### *Instances (1):*
File: [2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L86 )
```solidity
86: uint24 _poolFee,
```
### <a id=G2>[G-2]</a> Multiple accesses of a mapping/array should use a local variable cache.

##### Description
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata. [Similar findings](https://github.com/code-423n4/2022-06-infinity-findings/issues/186)

##### Recommendation
Use a local variable cache.

##### *Instances (7):*
File: [2023-03-asymmetry/contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L73 )
```solidity
73: (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74: derivatives[i].balance()) /
115: uint256 derivativeAmount = (derivatives[i].balance() *
118: derivatives[i].withdraw(derivativeAmount);
141: if (derivatives[i].balance() > 0)
142: derivatives[i].withdraw(derivatives[i].balance());
152: derivatives[i].deposit{value: ethAmount}();
```
### <a id=G3>[G-3]</a> `>=` costs less gas than `>`.

##### Description
The compiler uses opcodes `GT` and `ISZERO` for solidity code that uses `>`, but only requires `LT` for `>=`, which [saves 3 gas](https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde)

##### Recommendation
Use `>=` instead if appropriate.

##### *Instances (2):*
File: [2023-03-asymmetry/contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L141 )
```solidity
141: if (derivatives[i].balance() > 0)
```
File: [2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L200 )
```solidity
200: require(rethBalance2 > rethBalance1, "No rETH was minted");
```
### <a id=G4>[G-4]</a> `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

##### Description
Using the addition/subtraction operator instead of plus-equals/minus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**

##### Recommendation
Use `<x> = <x> + <y>` or `<x> = <x> - <y>` instead.

##### *Instances (4):*
File: [2023-03-asymmetry/contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L72 )
```solidity
72: underlyingValue +=
95: totalStakeValueEth += derivativeReceivedEthValue;
172: localTotalWeight += weights[i];
192: localTotalWeight += weights[i];
```
### <a id=G5>[G-5]</a> Using fixed bytes is cheaper than using  `string`.

##### Description
As a rule of thumb, use bytes for arbitrary-length raw byte data and `string` for arbitrary-length string (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of `bytes1` to `bytes32` because they are much cheaper.

##### Recommendation
Use fixed bytes instead of `string`.

##### *Instances (5):*
File: [2023-03-asymmetry/contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L49 )
```solidity
49: string memory _tokenName,
50: string memory _tokenSymbol
```
File: [2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L50 )
```solidity
50: function name() public pure returns (string memory) {
```
File: [2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L44 )
```solidity
44: function name() public pure returns (string memory) {
```
File: [2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L41 )
```solidity
41: function name() public pure returns (string memory) {
```
