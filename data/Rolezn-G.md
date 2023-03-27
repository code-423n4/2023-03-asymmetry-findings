## Summary<a name="Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | Setting the `constructor` to `payable` | 4 | 52 |
| [GAS&#x2011;2](#GAS&#x2011;2) | Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function | 2 | 56 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Empty Blocks Should Be Removed Or Emit Something | 4 | - |
| [GAS&#x2011;4](#GAS&#x2011;4) | Using `delete` statement can save gas | 3 | - |
| [GAS&#x2011;5](#GAS&#x2011;5) | Functions guaranteed to revert when called by normal users can be marked `payable` | 17 | 357 |
| [GAS&#x2011;6](#GAS&#x2011;6) | Use hardcoded address instead `address(this)` | 22 | - |
| [GAS&#x2011;7](#GAS&#x2011;7) | Optimize names to save gas | 3 | 66 |
| [GAS&#x2011;8](#GAS&#x2011;8) | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables | 4 | - |
| [GAS&#x2011;9](#GAS&#x2011;9) | Public Functions To External | 9 | - |
| [GAS&#x2011;10](#GAS&#x2011;10) | Save gas with the use of specific import statements | 22 | - |
| [GAS&#x2011;11](#GAS&#x2011;11) | Using `unchecked` blocks to save gas | 6 | 120 |
| [GAS&#x2011;12](#GAS&#x2011;12) | Use functions instead of modifiers | 1 | 100 |
| [GAS&#x2011;13](#GAS&#x2011;13) | Use solidity version 0.8.19 to gain some gas boost | 4 | 352 |
| [GAS&#x2011;14](#GAS&#x2011;14) | Save loop calls | 3 | - |

Total: 104 contexts over 14 issues

## Gas Optimizations


### <a href="#Summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> Setting the `constructor` to `payable`

Saves ~13 gas per instance

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





#### <a href="#Summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function

Saves deployment costs

#### <ins>Proof Of Concept</ins>

```solidity
66: require(sent, "Failed to send Ether");
77: require(sent, "Failed to send Ether");

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L66

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L77








### <a href="#Summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Empty Blocks Should Be Removed Or Emit Something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})

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






### <a href="#Summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Using `delete` statement can save gas

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






### <a href="#Summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### <ins>Proof Of Concept</ins>

```solidity
138: function rebalanceToWeights() external onlyOwner {

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L138

```solidity
165: function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L165

```solidity
182: function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L182

```solidity
202: function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
    ) external onlyOwner {

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
107: function withdraw(uint256 amount) external onlyOwner {

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L107

```solidity
156: function deposit() external payable onlyOwner returns (uint256) {

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L156

```solidity
51: function setMaxSlippage(uint256 _slippage) external onlyOwner {

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L51

```solidity
60: function withdraw(uint256 _amount) external onlyOwner {

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L60

```solidity
94: function deposit() external payable onlyOwner returns (uint256) {

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L94

```solidity
48: function setMaxSlippage(uint256 _slippage) external onlyOwner {

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L48

```solidity
56: function withdraw(uint256 _amount) external onlyOwner {

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L56

```solidity
73: function deposit() external payable onlyOwner returns (uint256) {

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L73



#### <ins>Recommended Mitigation Steps</ins>
Functions guaranteed to revert when called by normal users can be marked payable.



### <a href="#Summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry's script.sol and solmate's `LibRlp.sol` contracts can help achieve this.

References: 
https://book.getfoundry.sh/reference/forge-std/compute-create-address 

https://twitter.com/transmissions11/status/1518507047943245824

#### <ins>Proof Of Concept</ins>

```solidity
111: uint256 ethAmountBefore = address(this).balance;
121: uint256 ethAmountAfter = address(this).balance;

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L111

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L121



```solidity
139: uint256 ethAmountBefore = address(this).balance;
144: uint256 ethAmountAfter = address(this).balance;

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L139

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L144



```solidity
96: recipient: address(this),

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L96

```solidity
110: (bool sent, ) = address(msg.sender).call{value: address(this).balance}(

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L110

```solidity
197: uint256 rethBalance1 = rocketTokenRETH.balanceOf(address(this));
199: uint256 rethBalance2 = rocketTokenRETH.balanceOf(address(this));

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L197

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L199



```solidity
222: return IERC20(rethAddress()).balanceOf(address(this));

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L222

```solidity
63: address(this),
63: address(this)
63: address(this)
84: (bool sent, ) = address(msg.sender).call{value: address(this).balance}(

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L63

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L63

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L63

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L84



```solidity
99: address(this)
101: frxETHMinterContract.submitAndDeposit{value: msg.value}(address(this));
99: address(this)

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L99

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L101

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L99



```solidity
123: return IERC20(SFRX_ETH_ADDRESS).balanceOf(address(this));

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L123

```solidity
58: uint256 stEthBal = IERC20(STETH_TOKEN).balanceOf(address(this));
63: (bool sent, ) = address(msg.sender).call{value: address(this).balance}(

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L58

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L63



```solidity
74: uint256 wstEthBalancePre = IWStETH(WST_ETH).balanceOf(address(this));
78: uint256 wstEthBalancePost = IWStETH(WST_ETH).balanceOf(address(this));

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L74

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L78



```solidity
94: return IERC20(WST_ETH).balanceOf(address(this));

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L94



#### <ins>Recommended Mitigation Steps</ins>

Use hardcoded `address`








### <a href="#Summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

See more <a href="https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92">here</a>

#### <ins>Proof Of Concept</ins>

All in-scope contracts

#### <ins>Recommended Mitigation Steps</ins>
Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas
For example, the function IDs in the Gauge.sol contract will be the most used; A lower method ID may be given.



### <a href="#Summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables

#### <ins>Proof Of Concept</ins>


```solidity
72: underlyingValue +=
95: totalStakeValueEth += derivativeReceivedEthValue;

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L72

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L95



```solidity
172: localTotalWeight += weights[i];

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L172

```solidity
192: localTotalWeight += weights[i];

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L192




### <a href="#Summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> Public Functions To External

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

#### <ins>Proof Of Concept</ins>


```solidity
function name() public pure returns (string memory) {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L50

```solidity
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L211

```solidity
function balance() public view returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L221

```solidity
function name() public pure returns (string memory) {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L44

```solidity
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L111

```solidity
function balance() public view returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L122

```solidity
function name() public pure returns (string memory) {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L41

```solidity
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L86

```solidity
function balance() public view returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L93






### <a href="#Summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Non-usage of specific imports

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





### <a href="#Summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Using `unchecked` blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isnâ€™t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an `unchecked` block

#### <ins>Proof Of Concept</ins>

```solidity
122: uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L122

```solidity
174: ((10 ** 18 - maxSlippage))) / 10 ** 18);
201: uint256 rethMinted = rethBalance2 - rethBalance1;

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L174

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L201



```solidity
75: (10 ** 18 - maxSlippage)) / 10 ** 18;

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L75

```solidity
60: uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L60

```solidity
79: uint256 wstEthAmount = wstEthBalancePost - wstEthBalancePre;

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L79





### <a href="#Summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> Use functions instead of modifiers

#### <ins>Proof Of Concept</ins>

```solidity
52: ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);

```

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L52



#### <ins>Recommended Mitigation Steps</ins>
Functions guaranteed to revert when called by normal users can be marked payable.





### <a href="#Summary">[GAS&#x2011;13]</a><a name="GAS&#x2011;13"> Use solidity version 0.8.19 to gain some gas boost

Upgrade to the latest solidity version 0.8.19 to get additional gas savings.
See latest release for reference: https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

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



### <a href="#Summary">[GAS&#x2011;14]</a><a name="GAS&#x2011;14"> Save loop calls

Instead of calling `derivatives[i]` 3 times in each loop for fetching data, it can be saved as a variable.

```solidity
for (uint i = 0; i < derivativeCount; i++)
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                    derivatives[i].balance()) /
                10 ** 18;
```
https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L71-L75


```solidity
        for (uint256 i = 0; i < derivativeCount; i++) {
            // withdraw a percentage of each asset based on the amount of safETH
            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
            if (derivativeAmount == 0) continue; // if derivative empty ignore
            derivatives[i].withdraw(derivativeAmount);
        }
```
https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L113-L119


```solidity
        for (uint i = 0; i < derivativeCount; i++) {
            if (derivatives[i].balance() > 0)
                derivatives[i].withdraw(derivatives[i].balance());
        }
```
https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L140-L143
