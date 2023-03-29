# QA Report
## Finding Summary

[**Low Severity**](#Low-Severity)
1. [**Compiler With Known Bug May Be Used**](#1-Compiler-With-Known-Bug-May-Be-Used)
2. [**minAmount maxAmount Input Sanitization**](#2-minAmount-maxAmount-Input-Sanitization)
3. [**Single Step Owner Transfer**](#3-Single-Step-Owner-Transfer)
4. [**Gas Grief If Relayed**](#4-Gas-Grief-If-Relayed)
5. [**Owner Can Bypass setPauseStaking**](#5-Owner-Can-Bypass-setPauseStaking)

[**Non-Critical**](#Non-Critical)
1. [**Explicit Variable Not Used**](#1-Explicit-Variable-Not-Used)
2. [**bytes.concat() Can Be Used Over abi.encodePacked()**](#2-bytesconcat-can-be-used-over-abiencodepacked)
3. [**Inconsistent Named Returns**](#3-Inconsistent-Named-Returns)
4. [**Spelling Mistake**](#4-Spelling-Mistake)
5. [**Unnecessary Collapsed Code**](#5-Unnecessary-Collapsed-Code)
6. [**Inconsistent Single Line If Style**](#6-Inconsistent-Single-Line-If-Style)
7. [**Inconsistent Trailing Period**](#7-inconsistent-trailing-period)
8. [**Inconsistent Comment Capitalization**](#8-Inconsistent-Comment-Capitalization)
9. [**Constant Not Used**](#9-Constant-Not-Used)
10. [**Inconsistent Exponentiation Style**](#10-Inconsistent-Exponentiation-Style)
11. [**Mimicked ERC20 Names**](#11-Mimicked-ERC20-Names)
12. [**ERC20 Token Recovery**](#12-ERC20-Token-Recovery)
13. [**Unbounded Compiler Version**](#13-Unbounded-Compiler-Version)

[**Style Guide Violations**](#Style-Guide-Violations)
1. [**Maximum Line Length**](#1-Maximum-Line-Length)
2. [**Order of Functions**](#2-Order-of-Functions)
3. [**Whitespace in Expressions**](#3-Whitespace-in-Expressions)
4. [**Function Declaration Style**](#4-Function-Declaration-Style)

# Low Severity

## 1. Compiler With Known Bug May Be Used

All contracts in scope ([ex](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2)) use a Solidity version of `^0.8.13`. There is [a known bug](https://medium.com/certora/overly-optimistic-optimizer-certora-bug-disclosure-2101e3f7994d) that is present in Solidity `0.8.13` as well as `0.8.14`. Consider using a Solidity version past `0.8.14`.

## 2. minAmount maxAmount Input Sanitization

When setting the state variables `minAmount` and `maxAmount` in [`SafEth.sol`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol), there is no input sanitization to ensure `minAmount <= maxAmount`. See [`setMaxAmount`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223-L226) and [`setMinAmount`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214-L217). 

```Solidity
214:    function setMinAmount(uint256 _minAmount) external onlyOwner {
215:        minAmount = _minAmount;
216:        emit ChangeMinAmount(minAmount);
217:    }
```
	
	
```Solidity
223:    function setMaxAmount(uint256 _minAmount) external onlyOwner {
224:        maxAmount = _maxAmount;
225:        emit ChangeMaxAmount(maxAmount);
226:    }
```
	
There is no logical reason `minAmount > maxAmount` should be true, and admin error may make it so.

Consider adding a require check to make sure `minAmount <= maxAmount` when changing either variable.

## 3. Single Step Owner Transfer

All contracts in scope use openzeppelin's `OwnableUpgradeable` contract ([ex](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L12)) which uses a single step owner transfer seen [here](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/OwnableUpgradeable.sol#L83-L87): 

```Solidity
83:    function _transferOwnership(address newOwner) internal virtual {
84:        address oldOwner = _owner;
85:        _owner = newOwner;
86:        emit OwnershipTransferred(oldOwner, newOwner);
87:    }
```

Single step owner transfers are prone to admin error.

## 4. Gas Grief If Relayed

When using `(bool sent, ) = address(msg.sender).call`, leaving out the returnData does not prevent it from being copied to memory. If transactions were relayed, gas griefing is possible. Consider using assembly to prevent such a grief.

*/contracts/SafEth/SafEth.sol*

```Solidity
124:    (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
```

*/contracts/SafEth/derivatives/WstEth.sol*

```Solidity
63:    (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
76:    (bool sent, ) = WST_ETH.call{value: msg.value}("");
```

*/contracts/SafEth/derivatives/SfrxEth.sol*

```Solidity
84:    (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
```

*/contracts/SafEth/derivatives/Reth.sol*

```Solidity
110:    (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
```

For more information please see [this](https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19) post.

## 5. Owner Can Bypass setPauseStaking

The [`setPauseStaking`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232) function is used to pause staking function which emits an event: 

```Solidity
232:    function setPauseStaking(bool _pause) external onlyOwner {
233:        pauseStaking = _pause;
234:        emit StakingPaused(pauseStaking);
235:    }
```

The `StakingPaused` event can help users listen for if the protocol has paused staking; however, the owner can bypass this. The owner can add a derivative that forces a reversion in it's `ethPerDerivative` function. The [`stake`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63) function seen below loops over all derivatives, if one reverts the function will always revert - hence pausing staking. 

```
63:    function stake() external payable {
                                      ...
70:        // Getting underlying value in terms of ETH for each derivative
71:        for (uint i = 0; i < derivativeCount; i++)
72:            underlyingValue +=
73:                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74:                    derivatives[i].balance()) /
75:                10 ** 18;
                                      ...
101:    }
```

If the protocol is paused in the manner descibed above the honesty of the protocol is impacted. 

# Non-Critical

## 1. Explicit Variable Not Used

As described in the [Solidity documentation](https://docs.soliditylang.org/en/v0.4.21/types.html#integers): 

> *"`uint` and `int` are aliases for `uint256` and `int256`, respectively".* 

There are moments in the codebase where `uint` is used instead of the explicit `uint256`. It is best to be explicit with variable names to lessen confusion. Consider replacing instances of `uint` with `uint256`.

*/contracts/SafEth/SafEth.sol*

```solidity
26:	event Staked(address indexed recipient, uint ethIn, uint safEthOut);
27:	event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);
28:	event WeightChange(uint indexed index, uint weight);
31:	uint weight,
32:	uint index
92:	uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
203:	uint _derivativeIndex,
204:	uint _slippage
```

## 2. bytes.concat() Can Be Used Over abi.encodePacked()

Consider using `bytes.concat()` instead of `abi.encodePacked()` in contracts with Solidity version >= `0.8.4`.

*/contracts/SafEth/derivatives/Reth.sol*

```solidity
70:	abi.encodePacked("contract.address", "rocketTokenRETH")
125:	abi.encodePacked("contract.address", "rocketDepositPool")
136:	abi.encodePacked(
162:	abi.encodePacked("contract.address", "rocketDepositPool")
191:	abi.encodePacked("contract.address", "rocketTokenRETH")
233:	abi.encodePacked("contract.address", "rocketTokenRETH")
```

## 3. Inconsistent Named Returns

Some functions use named returns and others do not. It is best for code clearity to keep a consistent style.

1. No contracts only have named returns (EX. `returns(uint256 foo)`).
2. The following contracts only have non-named returns (EX. `returns(uint256)`): [WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol), and [SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol).
3. The following contracts have both: [Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol).

## 4. Spelling Mistake

There is a spelling mistake in the codebase. Consider fixing all spelling mistakes.

*contracts/SafEth/SafEth.sol*

* The word `derivatives` is misspelled as [`derivates`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L160).

## 5. Unnecessary Collapsed Code

There are moments in the codebase where lines (if expanded) less than 120 characters are collapsed - thus not voiding the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-lengthhttps://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length). The code [here](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L20-L27) and [here](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L15-L19) are examples. Some instances of this may void the Solidity Style Guide (see later in the report). Consider expanding code that is less than 120 characters.

## 6. Inconsistent Single Line If Style

[This](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L79-L81) block and [this](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L212-L215) block use single line conditional statements; however, in both cases the `if` style differs from the `else` style. The `if` statements use a newline where the `else` statement does not. Consider using a single consistant style.

## 7. Inconsistent Trailing Period

Almost all comments do not have a trailing period, with the exception of [this](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L163) line and [this](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L180) line. Consider removing all trailing periods to maintain style.

## 8. Inconsistent Comment Capitalization

There is an inconsistency in the capitalization of comments. For example, [this](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L114) line is not capitilized, while [this](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L151) line is. Consider capitilizing all comments.

## 9. Constant Not Used

The value `10 ** 18` is used consistantly throughout the codebase. Consider using a constant `DECIMALS` in place of `10 ** 18`.

## 10. Inconsistent Exponentiation Style

The value `10 ** 18` is used in the style seen for almost all instances in the codebase ([ex](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215)). There is [one](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L241) instance where `10 ** 18` is in the style `1e18`. Consider changing [this](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L241) style to `10 ** 18` or all instances to `1e18`.

## 11. Mimicked ERC20 Names

The ERC20 Token names do not indicate they are a derivative contract and use the native token names. They should be more descriptive. Consider changing the token names to better reflect the use and not confuse users. 

*/contracts/SafEth/derivatives/WstEth.sol*

```Solidity
41:    function name() public pure returns (string memory) {
42:        return "Lido";
43:    }
```

*/contracts/SafEth/derivatives/SfrxEth.sol*

```Solidity
44:    function name() public pure returns (string memory) {
45:        return "Frax";
46:    }
```

*/contracts/SafEth/derivatives/Reth.sol*

```Solidity
50:    function name() public pure returns (string memory) {
51:        return "RocketPool";
52:    }
```

## 12. ERC20 Token Recovery

Consider adding a recovery function for Tokens that are accidently sent to the core contracts of the protocol. 

## 13. Unbounded Compiler Version

All contracts in scope use a compiler version of `^0.8.13`. No one knows what will come in future compiler versions < `0.9.0`, thus it is best to put an upper limit on the compiler version.

# Style Guide Violations

## 1. Maximum Line Length

Lines with greater length than 120 characters are used. The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-lengthhttps://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) suggests that all lines should be 120 characters or less in width.

The following lines are longer than 120 characters, it is suggested to shorten these lines:

*contracts/SafEth/SafEth.sol*
*  [160](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L160). 

*contracts/SafEth/derivatives/Reth.sol*
*  [142](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L142). 

## 2. Order of Functions

The Solidity Style Guide suggests the following function order: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private.

The following contracts are not compliant (examples are only to prove the functions are out of order NOT a full description): 

* [WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol): external functions are positioned after public functions.
* [SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol): external functions are positioned after public functions.
* [Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol): external functions are positioned after public functions.

## 3. Whitespace in Expressions

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#whitespace-in-expressions) recommends to 

> *"[a]void extraneous whitespace [i]mmediately inside parenthesis, brackets or braces, with the exception of single line function declarations"*. 

The Style Guide also recommends against spaces before semicolons. Consider removing spaces before semicolons.

*The following do follow Solidity style conventions, but technically void the style guide*. 

*/contracts/SafEth/derivatives/WstEth.sol*

```solidity
63:	(bool sent, ) = address(msg.sender).call{value: address(this).balance}(
76:	(bool sent, ) = WST_ETH.call{value: msg.value}("");
```

*/contracts/SafEth/derivatives/SfrxEth.sol*

```solidity
84:	(bool sent, ) = address(msg.sender).call{value: address(this).balance}(
```

*/contracts/SafEth/SafEth.sol*

```solidity
124:	(bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
```

*/contracts/SafEth/derivatives/Reth.sol*

```solidity
110:	(bool sent, ) = address(msg.sender).call{value: address(this).balance}(
240:	(uint160 sqrtPriceX96, , , , , , ) = pool.slot0();
```

## 4. Function Declaration Style

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#function-declaration) states that 

> *"[f]or short function declarations, it is recommended for the opening brace of the function body to be kept on the same line as the function declaration"*. 

It is not clear what length is exactly meant by "short" or "long". The [maximum line length](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-lengthhttps://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) of 120 characters may be an indication, and in that case any expanded function declaration under 120 characters should be on a single line.

The following functions in their respective contracts are not compliant by this standard: 

*contracts/SafEth/SafEth.sol*
* [`initialize`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L48), [`adjustWeight`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165), [`addDerivative`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182), [`setMaxSlippage`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202). 
