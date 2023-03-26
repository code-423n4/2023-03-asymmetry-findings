## 1. For modern and more readable code; update import usages

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces allow us to apply this rule better.

Recommendation
`import {contract1 , contract2} from "filename.sol";`


## 2. Missing event for critical parameter change

Events help non-contract tools to track changes, and events prevent users from being surprised by changes


```
58	function setMaxSlippage(uint256 _slippage) external onlyOwner {
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol
```
51	function setMaxSlippage(uint256 _slippage) external onlyOwner {
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol
```
48	function setMaxSlippage(uint256 _slippage) external onlyOwner {
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol


## 3. Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

Use (e.g. 1e6) rather than decimal literals (e.g. 1000000), for better code readability.

```
44	maxSlippage = (1 * 10 ** 16); // 1%
171	uint rethPerEth = (10 ** 36) / poolPrice();
173	uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
174	((10 ** 18 - maxSlippage))) / 10 ** 18);
214	RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
215	else return (poolPrice() * 10 ** 18) / (10 ** 18);
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol
```
54	minAmount = 5 * 10 ** 17;
55	maxAmount = 200 * 10 ** 18;
75	10 ** 18;
80	preDepositPrice = 10 ** 18; // initializes with a price of 1
81	else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
94	) * depositAmount) / 10 ** 18;
98	uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol
```
38	maxSlippage = (1 * 10 ** 16);
74	uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
75	(10 ** 18 - maxSlippage)) / 10 ** 18;
113	10 ** 18
115	return ((10 ** 18 * frxAmount) /
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol
```
35	maxSlippage = (1 * 10 ** 16);
60	uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
87	return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol


## 4. `_safeMint()` should be used rather than `_mint()` wherever possible

`_mint()` is [discouraged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271) in favor of `_safeMint()` which ensures that the recipient is either an EOA or implements `IERC721Receiver`. Both OpenZeppelin and solmate have versions of this function so that NFTs aren’t lost if they’re minted to contracts that cannot transfer them back out.

```
99       _mint(msg.sender, mintAmount);
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol


## 5. Use safeTransferOwnership instead of transferOwnership function

`transferOwnership` function is used to change Ownership

Use a 2 structure transferOwnership which is safer.
`safeTransferOwnership`, use it as it's more secure due to 2-stage ownership transfer.

```
43       _transferOwnership(_owner);
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol
```
53       _transferOwnership(msg.sender);
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol
```
37       _transferOwnership(_owner);
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol
```
34       _transferOwnership(_owner);
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol


## 6. Use Ownable2StepUpgradeable instead of OwnableUpgradeable contract

`transferOwnership` function is used to change Ownership from `OwnableUpgradeable.sol`.

There is another Openzeppelin Ownable contract (Ownable2StepUpgradeable.sol) has `transferOwnership` function, use is more secure due to 2-stage ownership transfer.

```
13       import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol
```
9        import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol
```
6        import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol
```
5        import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol


## 7. For functions, follow Solidity standard naming conventions

The code doesn’t follow Solidity’s standard naming convention,

- internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

- public and external functions : only mixedCase


https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L83-L89
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L120
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L228
 







