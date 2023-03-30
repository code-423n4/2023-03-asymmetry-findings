## [L-01] **A single-step ownership transfer pattern is dangerous**

Inheriting from OpenZeppelin's `OwnableUpgradeable` contract means you are using a single-step ownership transfer pattern. If an admin provides an incorrect address for the new owner this will result in none of the `onlyOwner` marked methods being callable again. The better way to do this is to use a two-step ownership transfer approach, where the new owner should first claim his new rights before they are transferred.

Use OpenZeppelin's `Ownable2StepUpgradeable` instead of `OwnableUpgradeable`

[https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol)

## [L-02] Owner can renounce Ownership

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L9](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L9)

Typically, the contract's owner is the account that deploys the contract. As a result, the `owner` is able to perform certain privileged activities.

The Openzeppelin's `OwnableUpgradeable` used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

`onlyOwner`functions:

```solidity
File: contracts\SafEth\SafEth.sol

function rebalanceToWeights() external onlyOwner { }
function adjustWeight(uint256 _derivativeIndex,uint256 _weight) external onlyOwner { }
function addDerivative(address _contractAddress,uint256 _weight) external onlyOwner { }
function setMaxSlippage(uint _derivativeIndex,uint _slippage) external onlyOwner { }
function setMinAmount(uint256 _minAmount) external onlyOwner { }
function setMaxAmount(uint256 _maxAmount) external onlyOwner { }
function setPauseStaking(bool _pause) external onlyOwner { }
function setPauseUnstaking(bool _pause) external onlyOwner { }
```

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L13](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L13)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L6](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L6)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L5](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L5)

`onlyOwner`functions:

```solidity
File: contracts\SafEth\derivatives\Reth.sol
File: contracts\SafEth\derivatives\SfrxEth.sol
File: contracts\SafEth\derivatives\WstEth.sol

function setMaxSlippage(uint256 _slippage) external onlyOwner { }
function withdraw(uint256 amount) external onlyOwner { }
function deposit() external payable onlyOwner returns (uint256) { }
```

## [L-03] `owner` - Single point of failure

### Severity

**Likelihood:** Low, because it requires a malicious/compromised admin

**Impact:** High, because it can brick the protocol

### Description

The protocol's `owner` currently wields an excessive amount of control. In the event of a hack or compromise, the protocol would **fail**. This is not just a centralization risk, it represents a major point of **failure for the protocol**. If a hacker were to attack the owner and gain control over their EOA, the protocol would fail, resulting in financial losses for all customers.
Consider using a role-based access control approach instead of a single admin role as well as a timelock for important admin actions.

## [L-04] weight == 0 not clear to user

No mechanism to add pause/unpause to derivatives except explicitly set `weight == 0` but it can be unclear to user.

## [L-05] Missing R**eentrancyGuard** `unstake()`

Because `unstake()` function makes low-level external call that has no limit to gas spending is a good practice to add ReentrancyGuard.  All throw there is no explicit reentrancy in `unstake()` function adding ReentrancyGuard protects against potential uncovered reentrancy attacks.

## [L-06] fixed slippage controlled by protocol

There is no logic to getting user options for slippage. Protocol control minOut of token that user will get.

## [L-07] `ethPerDerivative` can return 0

If the **`sqrtPriceX96`** value is too low, the **`poolPrice()`** function will return 0 due to rounding, and consequently, the **`ethPerDerivative`** function will also return 0.

```solidity
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
    if (poolCanDeposit(_amount))
        return
            RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
    else return (poolPrice() * 10 ** 18) / (10 ** 18);
}

function poolPrice() private view returns (uint256) {
		...
    return (sqrtPriceX96 * (uint(sqrtPriceX96)) * (1e18)) >> (96 * 2);
}
```

## [Q-1] remove duplication code `adjustWeight()` and `addDerivative()`

Functions have an identical code snippet

```solidity
File: contracts\SafEth\SafEth.sol

function adjustWeight(uint256 _derivativeIndex,uint256 _weight) external onlyOwner {
function addDerivative(address _contractAddress,uint256 _weight) external onlyOwner {

```

```solidity
weights[derivativeCount] = _weight;

uint256 localTotalWeight = 0;
for (uint256 i = 0; i < derivativeCount; i++)
    localTotalWeight += weights[i];
totalWeight = localTotalWeight;
```

## [Q-2] Avoid comparing a boolean variable directly with **`true`**
 or **`false`**.

```solidity
File: contracts\SafEth\SafEth.sol

require(pauseStaking == false, "staking is paused");
require(pauseUnstaking == false, "unstaking is paused");
```

## [Q-3] Code consistency `uint` or `uint256` etc

Be consistent when writing code. A lot of `uin`t instead of `uint256`

In the **`swapExactInputSingleHop(...)`** function, the return variable **`amountOut`** is declared inline, which differs from the rest of the code where variables are declared within the function body.

```solidity
File: contracts\SafEth\derivatives\Reth.sol
function swapExactInputSingleHop(...) private returns (uint256 amountOut) {
```

## [Q-4] No interface for SafETH.sol and SafEthStorage.sol

## [Q-5] Not using the latest version of OpenZeppelin from dependencies

The current version of OpenZeppelin contracts/contracts-upgradeable is `v4.8.2`

```jsx
File: package.json
{ 
	"@openzeppelin/contracts": "^4.8.0",
	"@openzeppelin/contracts-upgradeable": "^4.8.1",
}
```

## [Q-6] Implement some type of version counter that will be incremented automatically for contract upgrades

With the inherent upgradeability of Proxies, contracts can be upgraded numerous times, utilizing a methodical approach to track the version of each upgrade. I recommend incorporating an automatic version counter that increments whenever the contract undergoes an upgrade.

## [Q-7] ****Incomplete NatSpec Comments****

Missing `@return` in NatSpec 

```solidity
File: contracts\SafEth\derivatives\Reth.sol

function rethAddress() private view returns (address) {
function swapExactInputSingleHop(address _tokenIn,address _tokenOut,uint24 _poolFee,uint256 _amountIn,uint256 _minOut) private returns (uint256 amountOut) {
function poolCanDeposit(uint256 _amount) private view returns (bool) {
function deposit() external payable onlyOwner returns (uint256) {
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
function balance() public view returns (uint256) {
function poolPrice() private view returns (uint256) {
```

```solidity
File: contracts\SafEth\derivatives\SfrxEth.sol

function name() public pure returns (string memory) {
function deposit() external payable onlyOwner returns (uint256) {
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
function balance() public view returns (uint256) {
```

```solidity
File: contracts\SafEth\derivatives\WstEth.sol

function name() public pure returns (string memory) {
function setMaxSlippage(uint256 _slippage) external onlyOwner {
function deposit() external payable onlyOwner returns (uint256) {
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
function balance() public view returns (uint256) {
```

Missing `@param` in NatSpec 

```solidity
File: contracts\SafEth\derivatives\Reth.sol

function withdraw(uint256 amount) external onlyOwner {
```

```solidity
File: contracts\SafEth\derivatives\SfrxEth.sol

function setMaxSlippage(uint256 _slippage) external onlyOwner {
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

```solidity
File: contracts\SafEth\derivatives\WstEth.sol

function setMaxSlippage(uint256 _slippage) external onlyOwner {
function withdraw(uint256 _amount) external onlyOwner {
function ethPerDerivative(uint256 _amount) public view returns (uint256) {

```

## [Q-8] ****Missing NatSpec comments****

```solidity
File: contracts\SafEth\SafEth.sol
File: contracts\SafEth\derivatives\Reth.sol
File: contracts\SafEth\derivatives\SfrxEth.sol
File: contracts\SafEth\derivatives\WstEth.sol

receive() external payable {}
```

## [Q-9] `Function writing` that does not comply with the `Solidity Style Guide` 

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

[https://docs.soliditylang.org/en/v0.8.17/style-guide.html](https://docs.soliditylang.org/en/v0.8.17/style-guide.html)

Functions should be grouped according to their visibility and ordered:

- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- within a grouping, place the view and pure functions last

## [Q-10] ****For modern and more readable code update import usage****

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces to allow us to apply this rule better.

`import {contract1 , contract2} from "filename.sol";`

## [Q-11]  Long lines are not suitable for the ‘Solidity Style Guide’ 

It is generally recommended that lines in the source code should not exceed 80-120 characters. Today’s screens are much larger, so in some cases it makes sense to expand that. The lines above should be split when they reach that length, as the files will most likely be on GitHub and GitHub always uses a scrollbar when the length is more than 164 characters.

[why-is-80-characters-the-standard-limit-for-code-width](https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width)

### Recommendation

Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section.

```solidity
141:	);
142:	RocketDAOProtocolSettingsDepositInterface rocketDAOProtocolSettingsDeposit = RocketDAOProtocolSettingsDepositInterface(
143:	    rocketProtocolSettingsAddress
144: 	);
```

## [Q-12]  Proper use of get as a function name prefix 

Clear function names can increase readability. Follow a standard convertion function names such as using get for getter (view/pure) functions.

```solidity
File: contracts\SafEth\derivatives\Reth.sol

function rethAddress() private view returns (address) {
function poolCanDeposit(uint256 _amount) private view returns (bool) {
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
function balance() public view returns (uint256) {
function poolPrice() private view returns (uint256) {
```

```solidity
File: contracts\SafEth\derivatives\SfrxEth.sol

function ethPerDerivative(uint256 _amount) public view returns (uint256) {
function balance() public view returns (uint256) {
```

```solidity
File: contracts\SafEth\derivatives\WstEth.sol

function ethPerDerivative(uint256 _amount) public view returns (uint256) {
function balance() public view returns (uint256) {
```

## [Q-13]  Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10 ** 18) 

```solidity
File: contracts\SafEth\SafEth.sol

minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
underlyingValue += (derivatives[i].ethPerDerivative(derivatives[i].balance()) * derivatives[i].balance()) / 10 ** 18;
preDepositPrice = 10 ** 18; // initializes with a price of 1
else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
) * depositAmount) / 10 ** 18;
uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```

```solidity
File: contracts\SafEth\derivatives\Reth.sol

maxSlippage = (1 * 10 ** 16); // 1%
uint rethPerEth = (10 ** 36) / poolPrice();
uint256 minOut = ((rethPerEth * msg.value / 10 ** 18) * (10 ** 18 - maxSlippage)) / 10 ** 18;
else return (poolPrice() * 10 ** 18) / (10 ** 18);
```

```solidity
File: contracts\SafEth\derivatives\SfrxEth.sol

maxSlippage = (1 * 10 ** 16); // 1%
uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) * (10 ** 18 - maxSlippage)) / 10 ** 18;
uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(10 ** 18);
return ((10 ** 18 * frxAmount) / IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
```

```solidity
File: contracts\SafEth\derivatives\WstEth.sol

maxSlippage = (1 * 10 ** 16); // 1%
uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
```

## [Q-14] Unused imports

```solidity
File: contracts\SafEth\SafEth.sol

import "../interfaces/IWETH.sol";
import "../interfaces/uniswap/ISwapRouter.sol";
import "../interfaces/lido/IWStETH.sol";
import "../interfaces/lido/IstETH.sol";
```

```solidity
File: contracts\SafEth\derivatives\Reth.sol

import "../../interfaces/frax/IsFrxEth.sol";
```

## [Q-15] Event is missing `indexed` fields 

Each `event` should use three `indexed` fields if there are three or more fields

```solidity
File: contracts\SafEth\SafEth.sol

event SetMaxSlippage(uint256 indexed index, uint256 slippage);
event Staked(address indexed recipient, uint ethIn, uint safEthOut);
event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);
event WeightChange(uint indexed index, uint weight);
event DerivativeAdded(address indexed contractAddress,uint weight,uint index);
```

## [Q-16]  Need Fuzzing test 

All tests use fixed values and don’t coder edge cases.