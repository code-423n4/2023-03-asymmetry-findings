## Summary<a name="Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | IERC20 `approve()` Is Deprecated | 2 |
| [LOW&#x2011;2](#LOW&#x2011;2) | Missing Contract-existence Checks Before Low-level Calls | 5 |
| [LOW&#x2011;3](#LOW&#x2011;3) | Contracts are not using their OZ Upgradeable counterparts | 4 |
| [LOW&#x2011;4](#LOW&#x2011;4) | TransferOwnership Should Be Two Step | 4 |
| [LOW&#x2011;5](#LOW&#x2011;5) | Unused `receive()` Function Will Lock Ether In Contract  | 4 |
| [LOW&#x2011;6](#LOW&#x2011;6) | No Storage Gap For Upgradeable Contracts | 1 |
| [LOW&#x2011;7](#LOW&#x2011;7) | Upgrade OpenZeppelin Contract Dependency | 1 |
| [LOW&#x2011;8](#LOW&#x2011;8) | Use `safeTransferOwnership` instead of `transferOwnership` function | 4 |
| [LOW&#x2011;9](#LOW&#x2011;9) | Consider the case where totalsupply is 0 | 2 |


Total: 27 contexts over 9 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Add a timelock to critical functions | 8 |
| [NC&#x2011;2](#NC&#x2011;2) | Avoid Floating Pragmas: The Version Should Be Locked | 4 |
| [NC&#x2011;3](#NC&#x2011;3) | No need for `== true` or `== false` checks | 2 |
| [NC&#x2011;4](#NC&#x2011;4) | Constants in comparisons should appear on the left side | 3 |
| [NC&#x2011;5](#NC&#x2011;5) | Critical Changes Should Use Two-step Procedure | 8 |
| [NC&#x2011;6](#NC&#x2011;6) | Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function | 2 |
| [NC&#x2011;7](#NC&#x2011;7) | Event emit should emit a parameter | 1 |
| [NC&#x2011;8](#NC&#x2011;8) | Function writing that does not comply with the Solidity Style Guide  | 4 |
| [NC&#x2011;9](#NC&#x2011;9) | Imports can be grouped together | 31 |
| [NC&#x2011;10](#NC&#x2011;10) | NatSpec return parameters should be included in contracts | 1 |
| [NC&#x2011;11](#NC&#x2011;11) | No need to initialize uints to zero | 3 |
| [NC&#x2011;12](#NC&#x2011;12) | Initial value check is missing in Set Functions | 8 |
| [NC&#x2011;13](#NC&#x2011;13) | Missing event for critical parameter change | 3 |
| [NC&#x2011;14](#NC&#x2011;14) | Implementation contract may not be initialized | 4 |
| [NC&#x2011;15](#NC&#x2011;15) | NatSpec comments should be increased in contracts | 1 |
| [NC&#x2011;16](#NC&#x2011;16) | Non-usage of specific imports | 22 |
| [NC&#x2011;17](#NC&#x2011;17) | Use a more recent version of Solidity | 4 |
| [NC&#x2011;18](#NC&#x2011;18) | Public Functions Not Called By The Contract Should Be Declared External Instead | 2 |
| [NC&#x2011;19](#NC&#x2011;19) | Use `bytes.concat()` | 10 |
| [NC&#x2011;20](#NC&#x2011;20) | Interchangeable usage of `uint` and `uint256` | 3 |

Total: 124 contexts over 20 issues

## Low Risk Issues

### <a href="#Summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> IERC20 `approve()` Is Deprecated

`approve()` is subject to a known front-running attack. It is deprecated in favor of `safeIncreaseAllowance()` and `safeDecreaseAllowance()`. If only setting the initial allowance to the value that means infinite, `safeIncreaseAllowance()` can be used instead.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#IERC20-approve-address-uint256-

#### <ins>Proof Of Concept</ins>

```solidity
90: IERC20(_tokenIn).approve(UNISWAP_ROUTER, _amountIn);
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L90

```solidity
59: IERC20(STETH_TOKEN).approve(LIDO_CRV_POOL, stEthBal);
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L59



#### <ins>Recommended Mitigation Steps</ins>

Consider using `safeIncreaseAllowance()` / `safeDecreaseAllowance()` instead.






### <a href="#Summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> Missing Contract-existence Checks Before Low-level Calls

Low-level calls return success if there is no code present at the specified address. 

#### <ins>Proof Of Concept</ins>


```solidity
124: (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L124

```solidity
110: (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L110

```solidity
84: (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L84

```solidity
63: (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L63

```solidity
76: (bool sent, ) = WST_ETH.call{value: msg.value}("");
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L76




#### <ins>Recommended Mitigation Steps</ins>

In addition to the zero-address checks, add a check to verify that `<address>.code.length > 0`



### <a href="#Summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Contracts are not using their OZ Upgradeable counterparts

The non-upgradeable standard version of OpenZeppelin’s library are inherited / used by the contracts.
It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

#### <ins>Proof of Concept</ins>

```solidity
4: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L4

```solidity
6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L6

```solidity
7: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L7

```solidity
6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L6



#### <ins>Recommended Mitigation Steps</ins>

Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.
See https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/tree/master/contracts for list of available upgradeable contracts




### <a href="#Summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> TransferOwnership Should Be Two Step

Recommend considering implementing a two step process where the owner or admin nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.

#### <ins>Proof Of Concept</ins>


```solidity
53: function initialize: _transferOwnership(msg.sender);
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L53

```solidity
43: function initialize: _transferOwnership(_owner);
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L43

```solidity
37: function initialize: _transferOwnership(_owner);
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L37

```solidity
34: function initialize: _transferOwnership(_owner);
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L34



#### <ins>Recommended Mitigation Steps</ins>

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.



### <a href="#Summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Unused `receive()` Function Will Lock Ether In Contract 

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert

#### <ins>Proof Of Concept</ins>


```solidity
246: receive() external payable {}
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L246

```solidity
244: receive() external payable {}
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L244

```solidity
126: receive() external payable {}
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L126

```solidity
97: receive() external payable {}
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L97



#### <ins>Recommended Mitigation Steps</ins>

The function should call another function, otherwise it should revert



### <a href="#Summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> No storage gap for upgradeable contract might lead to storage slot collision

For upgradeable contracts, there must be storage gap to "allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments". Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts.

Refer to the bottom part of this article: https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable

#### <ins>Proof Of Concept</ins>

However, the contract doesn't contain a storage gap. The storage gap is essential for upgradeable contract because "It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments". See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

```solidity
contract SafEth is
    Initializable,
    ERC20Upgradeable,
    OwnableUpgradeable,
    SafEthStorage
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L5


#### <ins>Recommended Mitigation Steps</ins>
Recommend adding appropriate storage gap at the end of upgradeable contracts such as the below. Please reference OpenZeppelin upgradeable contract templates.
```solidity
    uint256[50] private __gap;
```



### <a href="#Summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> Upgrade OpenZeppelin Contract Dependency

An outdated OZ version is used (which has known vulnerabilities, see: https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories).

#### <ins>Proof Of Concept</ins>

Require dependencies to be at least version of 4.8.1 to include critical vulnerability patches.

```solidity
    "@openzeppelin/contracts": "^4.8.0"
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/package.json#L82



#### <ins>Recommended Mitigation Steps</ins>

Update OpenZeppelin Contracts Usage in package.json and require at least version of 4.8.1 via `>=4.8.1`



### <a href="#Summary">[LOW&#x2011;8]</a><a name="LOW&#x2011;8"> Use `safeTransferOwnership` instead of `transferOwnership` function

Use `safeTransferOwnership` which is safer. Use it as it is more secure due to 2-stage ownership transfer.

#### <ins>Proof Of Concept</ins>


```solidity
53: function initialize: _transferOwnership(msg.sender);
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L53

```solidity
43: function initialize: _transferOwnership(_owner);
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L43

```solidity
37: function initialize: _transferOwnership(_owner);
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L37

```solidity
34: function initialize: _transferOwnership(_owner);
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L34



#### <ins>Recommended Mitigation Steps</ins>

Use <a href="https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol">Ownable2Step.sol</a>

```solidity
    function acceptOwnership() external {
        address sender = _msgSender();
        require(pendingOwner() == sender, "Ownable2Step: caller is not the new owner");
        _transferOwnership(sender);
    }
}
```

### <a href="#Summary">[LOW&#x2011;9]</a><a name="LOW&#x2011;9"> Consider the case where totalsupply is 0

Consider the case where `totalsupply` is 0. When `totalsupply` is 0, it should return 0 directly, because there will be an error of dividing by 0.

#### <ins>Impact</ins>

This would cause the affected functions to revert and as a result can lead to potential loss.

#### <ins>Proof Of Concept</ins>

```solidity
81: else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L81

```solidity
116: _safEthAmount) / safEthTotalSupply;

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L116


#### <ins>Recommended Mitigation Steps</ins>
Add check for zero value and return 0.

```solidity
if ( totalSupply() == 0) return 0;
```


## Non Critical Issues

### <a href="#Summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). Consider adding a timelock to the following functions:

#### <ins>Proof Of Concept</ins>


```solidity
202: function setMaxSlippage(
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L202

```solidity
214: function setMinAmount(uint256 _minAmount) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L214

```solidity
223: function setMaxAmount(uint256 _maxAmount) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L223

```solidity
232: function setPauseStaking(bool _pause) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L232

```solidity
241: function setPauseUnstaking(bool _pause) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L241

```solidity
58: function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L58

```solidity
51: function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L51

```solidity
48: function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L48







### <a href="#Summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Avoid Floating Pragmas: The Version Should Be Locked

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

#### <ins>Proof Of Concept</ins>

```solidity
Found usage of floating pragmas ^0.8.13 of Solidity in [SafEth.sol]
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L2

```solidity
Found usage of floating pragmas ^0.8.13 of Solidity in [Reth.sol]
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L2

```solidity
Found usage of floating pragmas ^0.8.13 of Solidity in [SfrxEth.sol]
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L2

```solidity
Found usage of floating pragmas ^0.8.13 of Solidity in [WstEth.sol]
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L2






### <a href="#Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> No need for `== true` or `== false` checks

There is no need to verify that `== true` or `== false` when the variable checked upon is a boolean as well.

#### <ins>Proof Of Concept</ins>


```solidity
64: require(pauseStaking == false, "staking is paused");

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L64

```solidity
109: require(pauseUnstaking == false, "unstaking is paused");

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L109




#### <ins>Recommended Mitigation Steps</ins>

Instead simply check for `variable` or `!variable`





### <a href="#Summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Constants in comparisons should appear on the left side

Doing so will prevent <a href="https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html">typo bugs</a>

#### <ins>Proof Of Concept</ins>

```solidity
79: if (totalSupply == 0)
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L79

```solidity
87: if (weight == 0) continue;
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L87

```solidity
117: if (derivativeAmount == 0) continue;
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L117






### <a href="#Summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Critical Changes Should Use Two-step Procedure

The critical procedures should be two step process.

See similar findings in previous Code4rena contests for reference:
https://code4rena.com/reports/2022-06-illuminate/#2-critical-changes-should-use-two-step-procedure

#### <ins>Proof Of Concept</ins>

```solidity
202: function setMaxSlippage(
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L202

```solidity
214: function setMinAmount(uint256 _minAmount) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L214

```solidity
223: function setMaxAmount(uint256 _maxAmount) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L223

```solidity
232: function setPauseStaking(bool _pause) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L232

```solidity
241: function setPauseUnstaking(bool _pause) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L241

```solidity
58: function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L58

```solidity
51: function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L51

```solidity
48: function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L48



#### <ins>Recommended Mitigation Steps</ins>

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.



#### <a href="#Summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function

Saves deployment costs

#### <ins>Proof Of Concept</ins>

```solidity
66: require(sent, "Failed to send Ether");
77: require(sent, "Failed to send Ether");

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L66

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L77








### <a href="#Summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> Event emit should emit a parameter

Some emitted events do not have any emitted parameters. It is recommended to add some parameter such as state changes or value changes when events are emitted

#### <ins>Proof Of Concept</ins>


```solidity
154: emit Rebalanced()

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L154






### <a href="#Summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Function writing that does not comply with the Solidity Style Guide

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:
- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- within a grouping, place the view and pure functions last

#### <ins>Proof Of Concept</ins>

See `Reth.sol` for example




### <a href="#Summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> Imports can be grouped together

Consider importing OZ first, then all interfaces, then all utils.

#### <ins>Proof Of Concept</ins>


```solidity
4: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
5: import "../interfaces/IWETH.sol";
6: import "../interfaces/uniswap/ISwapRouter.sol";
7: import "../interfaces/lido/IWStETH.sol";
8: import "../interfaces/lido/IstETH.sol";
9: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
10: import "./SafEthStorage.sol";
11: import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L4-L11

```solidity
4: import "../../interfaces/IDerivative.sol";
5: import "../../interfaces/frax/IsFrxEth.sol";
6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
7: import "../../interfaces/rocketpool/RocketStorageInterface.sol";
8: import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
9: import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
10: import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
11: import "../../interfaces/IWETH.sol";
12: import "../../interfaces/uniswap/ISwapRouter.sol";
13: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
14: import "../../interfaces/uniswap/IUniswapV3Factory.sol";
15: import "../../interfaces/uniswap/IUniswapV3Pool.sol";

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L4-L15

```solidity
4: import "../../interfaces/IDerivative.sol";
5: import "../../interfaces/frax/IsFrxEth.sol";
6: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
7: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
8: import "../../interfaces/curve/IFrxEthEthPool.sol";
9: import "../../interfaces/frax/IFrxETHMinter.sol";

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L4-L9

```solidity
4: import "../../interfaces/IDerivative.sol";
5: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
7: import "../../interfaces/curve/IStEthEthPool.sol";
8: import "../../interfaces/lido/IWStETH.sol";

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L4-L8







### <a href="#Summary">[NC&#x2011;10]</a><a name="NC&#x2011;10"> NatSpec return parameters should be included in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html
#### <ins>Proof Of Concept</ins>


All in-scope contracts


#### <ins>Recommended Mitigation Steps</ins>

Include return parameters in NatSpec comments

Recommendation Code Style: (from Uniswap3)

```solidity
    /// @notice Adds liquidity for the given recipient/tickLower/tickUpper position
    /// @dev The caller of this method receives a callback in the form of IUniswapV3MintCallback#uniswapV3MintCallback
    /// in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
    /// on tickLower, tickUpper, the amount of liquidity, and the current price.
    /// @param recipient The address for which the liquidity will be created
    /// @param tickLower The lower tick of the position in which to add liquidity
    /// @param tickUpper The upper tick of the position in which to add liquidity
    /// @param amount The amount of liquidity to mint
    /// @param data Any data that should be passed through to the callback
    /// @return amount0 The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback
    /// @return amount1 The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback
    function mint(
        address recipient,
        int24 tickLower,
        int24 tickUpper,
        uint128 amount,
        bytes calldata data
    ) external returns (uint256 amount0, uint256 amount1);
```



### <a href="#Summary">[NC&#x2011;11]</a><a name="NC&#x2011;11"> No need to initialize uints to zero

There is no need to initialize `uint` variables to zero as their default value is `0`

#### <ins>Proof Of Concept</ins>

```solidity
68: uint256 underlyingValue = 0;

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L68

```solidity
170: uint256 localTotalWeight = 0;

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L170

```solidity
190: uint256 localTotalWeight = 0;

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L190





### <a href="#Summary">[NC&#x2011;12]</a><a name="NC&#x2011;12"> Initial value check is missing in Set Functions

A check regarding whether the current value and the new value are the same should be added

#### <ins>Proof Of Concept</ins>

```solidity
202: function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
    ) external onlyOwner {
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
    }
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L202

```solidity
214: function setMinAmount(uint256 _minAmount) external onlyOwner {
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L214

```solidity
223: function setMaxAmount(uint256 _maxAmount) external onlyOwner {
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
    }
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L223

```solidity
232: function setPauseStaking(bool _pause) external onlyOwner {
        pauseStaking = _pause;
        emit StakingPaused(pauseStaking);
    }
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L232

```solidity
241: function setPauseUnstaking(bool _pause) external onlyOwner {
        pauseUnstaking = _pause;
        emit UnstakingPaused(pauseUnstaking);
    }

    receive() external payable {}
}
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L241

```solidity
58: function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L58

```solidity
51: function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L51

```solidity
48: function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L48






### <a href="#Summary">[NC&#x2011;13]</a><a name="NC&#x2011;13"> Missing event for critical parameter change

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

#### <ins>Proof Of Concept</ins>


```solidity
58: function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L58

```solidity
51: function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L51

```solidity
48: function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L48





### <a href="#Summary">[NC&#x2011;14]</a><a name="NC&#x2011;14"> Implementation contract may not be initialized

OpenZeppelin recommends that the initializer modifier be applied to constructors. 
Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.
https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

#### <ins>Proof Of Concept</ins>


```solidity
38: constructor()
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L38

```solidity
33: constructor()
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L33

```solidity
27: constructor()
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L27

```solidity
24: constructor()
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L24





### <a href="#Summary">[NC&#x2011;15]</a><a name="NC&#x2011;15"> NatSpec comments should be increased in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

#### <ins>Proof Of Concept</ins>


All in-scope contracts


#### <ins>Recommended Mitigation Steps</ins>

NatSpec comments should be increased in contracts



### <a href="#Summary">[NC&#x2011;16]</a><a name="NC&#x2011;16"> Non-usage of specific imports

The current form of relative path import is not recommended for use because it can unpredictably pollute the namespace.
Instead, the Solidity docs recommend specifying imported symbols explicitly.
https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files

A good example:
```solidity
import {OwnableUpgradeable} from "openzeppelin-contracts-upgradeable/contracts/access/OwnableUpgradeable.sol";
import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
import {SafeCastLib} from "solmate/utils/SafeCastLib.sol";
import {ERC20} from "solmate/tokens/ERC20.sol";
import {IProducer} from "src/interfaces/IProducer.sol";
import {GlobalState, UserState} from "src/Common.sol";
```

#### <ins>Proof Of Concept</ins>


```solidity
5: import "../interfaces/IWETH.sol";
6: import "../interfaces/uniswap/ISwapRouter.sol";
7: import "../interfaces/lido/IWStETH.sol";
8: import "../interfaces/lido/IstETH.sol";
10: import "./SafEthStorage.sol";

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L5

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L6

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L7

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L8

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L10



```solidity
4: import "../../interfaces/IDerivative.sol";
5: import "../../interfaces/frax/IsFrxEth.sol";
7: import "../../interfaces/rocketpool/RocketStorageInterface.sol";
8: import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
9: import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
10: import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
11: import "../../interfaces/IWETH.sol";
12: import "../../interfaces/uniswap/ISwapRouter.sol";
14: import "../../interfaces/uniswap/IUniswapV3Factory.sol";
15: import "../../interfaces/uniswap/IUniswapV3Pool.sol";

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L4

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L5

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L7

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L8

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L9

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L10

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L11

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L12

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L14

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L15



```solidity
4: import "../../interfaces/IDerivative.sol";
5: import "../../interfaces/frax/IsFrxEth.sol";
8: import "../../interfaces/curve/IFrxEthEthPool.sol";
9: import "../../interfaces/frax/IFrxETHMinter.sol";

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L4

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L5

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L8

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L9



```solidity
4: import "../../interfaces/IDerivative.sol";
7: import "../../interfaces/curve/IStEthEthPool.sol";
8: import "../../interfaces/lido/IWStETH.sol";

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L4

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L7

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L8






#### <ins>Recommended Mitigation Steps</ins>

Use specific imports syntax per solidity docs recommendation.



### <a href="#Summary">[NC&#x2011;17]</a><a name="NC&#x2011;17"> Use a more recent version of Solidity

<a href="https://blog.soliditylang.org/2022/05/18/solidity-0.8.14-release-announcement/">0.8.14</a>:

ABI Encoder: When ABI-encoding values from calldata that contain nested arrays, correctly validate the nested array length against calldatasize() in all cases.
Override Checker: Allow changing data location for parameters only when overriding external functions.

<a href="https://blog.soliditylang.org/2022/06/15/solidity-0.8.15-release-announcement/">0.8.15</a>:

Code Generation: Avoid writing dirty bytes to storage when copying bytes arrays.
Yul Optimizer: Keep all memory side-effects of inline assembly blocks.

<a href="https://blog.soliditylang.org/2022/08/08/solidity-0.8.16-release-announcement/">0.8.16</a>:

Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.

<a href="https://blog.soliditylang.org/2022/09/08/solidity-0.8.17-release-announcement/">0.8.17</a>:

Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.

<a href="https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/">0.8.19</a>:
SMTChecker: New trusted mode that assumes that any compile-time available code is the actual used code, even in external calls. 
Bug Fixes: 
- Assembler: Avoid duplicating subassembly bytecode where possible.
- Code Generator: Avoid including references to the deployed label of referenced functions if they are called right away.
- ContractLevelChecker: Properly distinguish the case of missing base constructor arguments from having an unimplemented base function.
- SMTChecker: Fix internal error caused by unhandled z3 expressions that come from the solver when bitwise operators are used.
- SMTChecker: Fix internal error when using the custom NatSpec annotation to abstract free functions.
- TypeChecker: Also allow external library functions in using for.

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L2

```solidity
pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L2

```solidity
pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L2

```solidity
pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L2



#### <ins>Recommended Mitigation Steps</ins>

Consider updating to a more recent solidity version.



### <a href="#Summary">[NC&#x2011;18]</a><a name="NC&#x2011;18"> Public Functions Not Called By The Contract Should Be Declared External Instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public.

#### <ins>Proof Of Concept</ins>


```solidity
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L211

```solidity
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L86





### <a href="#Summary">[NC&#x2011;19]</a><a name="NC&#x2011;19"> Use `bytes.concat()`

Solidity version 0.8.4 introduces `bytes.concat()` (vs `abi.encodePacked(<bytes>,<bytes>)`)

#### <ins>Proof Of Concept</ins>


```solidity
70: abi.encodePacked("contract.address", "rocketTokenRETH")
                )
            )
233: abi.encodePacked("contract.address", "rocketTokenRETH")
                )
            )
125: abi.encodePacked("contract.address", "rocketDepositPool")
                )
            )
162: abi.encodePacked("contract.address", "rocketDepositPool")
                )
            )
136: abi.encodePacked(
                        "contract.address",
                        "rocketDAOProtocolSettingsDeposit"
                    )
                )
            )
125: abi.encodePacked("contract.address", "rocketDepositPool")
                )
            )
162: abi.encodePacked("contract.address", "rocketDepositPool")
                )
            )
191: abi.encodePacked("contract.address", "rocketTokenRETH")
                    )
                )
70: abi.encodePacked("contract.address", "rocketTokenRETH")
                )
            )
233: abi.encodePacked("contract.address", "rocketTokenRETH")
                )
            )

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L70

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L233

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L125

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L162

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L136

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L125

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L162

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L191

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L70

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L233






#### <ins>Recommended Mitigation Steps</ins>

Use `bytes.concat()` and upgrade to at least Solidity version 0.8.4 if required. 





### <a href="#Summary">[NC&#x2011;20]</a><a name="NC&#x2011;20"> Interchangeable usage of `uint` and `uint256`

Some developers prefer to use `uint256` because it is consistent with other `uint` data types, which also specify their size, and also because making the size of the data explicit reminds the developer and the reader how much data they've got to play with, which may help prevent or detect bugs.

For example, if doing bytes4(keccak('transfer(address, uint)’)), you'll get a different method sig ID than bytes4(keccak('transfer(address, uint256)’)) and smart contracts will only understand the latter when comparing method sig IDs

#### <ins>Proof Of Concept</ins>


```solidity
setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
    ) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L202

```solidity
adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L165

```solidity
addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L182








