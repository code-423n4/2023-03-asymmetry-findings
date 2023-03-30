# Low Summary
| |Issue|Instances|
|:-:|:-|:-:|
|[L-01]|Use two-step ownership transfers|4|
|[L-02]|Sanitise inputs for critical parameter changes|9|
|[L-03]|Use timelock for critical parameter changes|8|

Total Issues: 3

Total instances: 21

&nbsp;
# Non-Critical Summary
| |Issue|Instances|
|:-:|:-|:-:|
|[N-01]|Use `indexed` for event parameters|6|
|[N-02]|Use fixed compiler version|4|
|[N-03]|Use appropriate function naming convention|4|
|[N-04]|Update import usages|31|
|[N-05]|Implementing `renounceOwnership` is dangerous|4|
|[N-06]|Remove unused imports|6|
|[N-07]|Missing `address(0)` checks in constructor/initialize|3|
|[N-08]|Use built in constants over magic numbers|3|
|[N-09]|Typo in comments|1|

Total issues: 9

Total instances: 62

&nbsp;
# Low Risk Issues
## [L-01] Use two-step ownership transfers

The current ownership transfer process involves the current owner calling `transferOwnership()`. This function checks the new owner is not the zero address and proceeds to write the new owner's address into the owner's state variable.

If the nominated EOA account is not a valid account, it is entirely possible the owner may accidentally transfer ownership to an uncontrolled account, breaking all functions with the onlyOwner() modifier.

Consider implementing a two step process where the owner nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed.

This can be achieved using OpenZeppelin's `Ownable2Step` or `Ownable2StepUpgradeable`.

Instances: 4
```solidity
File: contracts/SafEth/derivatives/Reth.sol

19: contract Reth is IDerivative, Initializable, OwnableUpgradeable {

```
- [contracts/SafEth/derivatives/Reth.sol#L19](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L19)

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

12: contract WstEth is IDerivative, Initializable, OwnableUpgradeable {

```
- [contracts/SafEth/derivatives/WstEth.sol#L12](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L12)

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

13: contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {

```
- [contracts/SafEth/derivatives/SfrxEth.sol#L13](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L13)

```solidity
File: contracts/SafEth/SafEth.sol

18:     OwnableUpgradeable,

```
- [contracts/SafEth/SafEth.sol#L18](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/safeth/SafEth.sol#L18)

&nbsp;
## [L-02] Sanitise inputs for critical parameter changes

Omitting checks on function parameters for `onlyOwner` functions that change critical state variables may lead to DoS or lost funds if an erroneous argument is passed.

Instances: 9
```solidity
File: contracts/SafEth/SafEth.sol

167:        uint256 _weight

183:        address _contractAddress,

184:        uint256 _weight

204:        uint _slippage

214:    function setMinAmount(uint256 _minAmount) external onlyOwner {

223:    function setMaxAmount(uint256 _maxAmount) external onlyOwner {

```
- [contracts/SafEth/SafEth.sol#L167](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L167)
- [contracts/SafEth/SafEth.sol#L183](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L183)
- [contracts/SafEth/SafEth.sol#L184](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L184)
- [contracts/SafEth/SafEth.sol#L204](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L204)
- [contracts/SafEth/SafEth.sol#L214](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214)
- [contracts/SafEth/SafEth.sol#L223](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223)

```solidity
File: contracts/SafEth/derivatives/Reth.sol

58:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

```
- [contrats/SafEth/derivatives/Reth.sol#L58](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58)

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

51:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

```
- [contrats/SafEth/derivatives/SfrxEth.sol#L51](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51)

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

```
- [contrats/SafEth/derivatives/WstEth.sol#L48](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48)

&nbsp;
## [L-03] Use timelock for critical parameter changes

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

Instances: 8
```solidity
File: contracts/SafEth/SafEth.sol

165:    function adjustWeight(

182:    function addDerivative(

202:    function setMaxSlippage(

214:    function setMinAmount(uint256 _minAmount) external onlyOwner {

223:    function setMaxAmount(uint256 _maxAmount) external onlyOwner {

```
- [contracts/safeth/SafEth.sol#L165](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165)
- [contracts/safeth/SafEth.sol#L182](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182)
- [contracts/safeth/SafEth.sol#L202](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202)
- [contracts/safeth/SafEth.sol#L214](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214)
- [contracts/safeth/SafEth.sol#L223](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223)

```solidity
File: contracts/SafEth/derivatives/Reth.sol

58:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

```
- [contrats/SafEth/derivatives/Reth.sol#L58](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58)

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

51:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

```
- [contrats/SafEth/derivatives/SfrxEth.sol#L51](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51)

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

```
- [contrats/SafEth/derivatives/WstEth.sol#L48](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48)

&nbsp;

&nbsp;

# Non-Critical Issues
## [N-01] Use `indexed` for event parameters

Index event fields make the field more quickly accessible to off-chain tools that parse events. 

If the variable is value type (uint, address, bool), then using `indexed` saves gas. Otherwise, each index field costs extra gas during emission.

Instances: 6
```solidity
File: contracts/SafEth/SafEth.sol

25:     event SetMaxSlippage(uint256 indexed index, uint256 slippage);

26:     event Staked(address indexed recipient, uint ethIn, uint safEthOut);

27:     event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);

28:     event WeightChange(uint indexed index, uint weight);

31:         uint weight,

32:         uint index

```
- [contracts/SafEth/SafEth.sol#L25](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L25)
- [contracts/SafEth/SafEth.sol#L26](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L26)
- [contracts/SafEth/SafEth.sol#L27](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L27)
- [contracts/SafEth/SafEth.sol#L28](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L28)
- [contracts/SafEth/SafEth.sol#L31](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L31)
- [contracts/SafEth/SafEth.sol#L32](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L32)

&nbsp;
## [N-02] Use fixed compiler version

A floating pragma risks a different compiler version being used in production vs testing, which poses security risks.

Instances: 4
```solidity
File: contracts/SafEth/derivatives/Reth.sol

2: pragma solidity ^0.8.13;

```
- [contracts/SafEth/derivatives/Reth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2)

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

2: pragma solidity ^0.8.13;

```
- [contracts/SafEth/derivatives/WstEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L2)

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

2: pragma solidity ^0.8.13;

```
- [contracts/SafEth/derivatives/SfrxEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L2)

```solidity
File: contracts/SafEth/SafEth.sol

2: pragma solidity ^0.8.13;

```
- [contracts/SafEth/SafEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2)

&nbsp;
## [N-03] Use appropriate function naming convention

According to Solidity's style guide and popular convention, function names should be prefixed with an underscore if and only if they are `private` or `internal`.

See [here](https://primitivefi.notion.site/Solidity-Style-44daebebfbd645b0b9cbad7075ba42fe) for more details.


Instances: 4
```solidity
File: contracts/SafEth/derivatives/Reth.sol

66:     function rethAddress() private view returns (address) {

83:     function swapExactInputSingleHop(

120:     function poolCanDeposit(uint256 _amount) private view returns (bool) {

228:     function poolPrice() private view returns (uint256) {

```
- [contracts/SafEth/derivatives/Reth.sol#L66](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L66)
- [contracts/SafEth/derivatives/Reth.sol#L83](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L83)
- [contracts/SafEth/derivatives/Reth.sol#L120](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L120)
- [contracts/SafEth/derivatives/Reth.sol#L228](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L228)

&nbsp;
## [N-04] Update import usages

To improve readability and adhere to the rule of modularity and modular programming: only import what you need. Specific imports with curly braces allow us to apply this rule better.

For example:
```
import {ERC721} from "solmate/tokens/ERC721.sol";
```

Instances: 31
```solidity
File: contracts/SafEth/derivatives/Reth.sol

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
- [contracts/SafEth/derivatives/Reth.sol#L4](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L4)
- [contracts/SafEth/derivatives/Reth.sol#L5](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L5)
- [contracts/SafEth/derivatives/Reth.sol#L6](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L6)
- [contracts/SafEth/derivatives/Reth.sol#L7](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L7)
- [contracts/SafEth/derivatives/Reth.sol#L8](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L8)
- [contracts/SafEth/derivatives/Reth.sol#L9](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L9)
- [contracts/SafEth/derivatives/Reth.sol#L10](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L10)
- [contracts/SafEth/derivatives/Reth.sol#L11](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L11)
- [contracts/SafEth/derivatives/Reth.sol#L12](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L12)
- [contracts/SafEth/derivatives/Reth.sol#L13](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L13)
- [contracts/SafEth/derivatives/Reth.sol#L14](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L14)
- [contracts/SafEth/derivatives/Reth.sol#L15](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L15)

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

4: import "../../interfaces/IDerivative.sol";

5: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

7: import "../../interfaces/curve/IStEthEthPool.sol";

8: import "../../interfaces/lido/IWStETH.sol";

```
- [contracts/SafEth/derivatives/WstEth.sol#L4](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L4)
- [contracts/SafEth/derivatives/WstEth.sol#L5](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L5)
- [contracts/SafEth/derivatives/WstEth.sol#L6](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L6)
- [contracts/SafEth/derivatives/WstEth.sol#L7](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L7)
- [contracts/SafEth/derivatives/WstEth.sol#L8](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L8)

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

4: import "../../interfaces/IDerivative.sol";

5: import "../../interfaces/frax/IsFrxEth.sol";

6: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

7: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

8: import "../../interfaces/curve/IFrxEthEthPool.sol";

9: import "../../interfaces/frax/IFrxETHMinter.sol";

```
- [contracts/SafEth/derivatives/SfrxEth.sol#L4](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L4)
- [contracts/SafEth/derivatives/SfrxEth.sol#L5](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L5)
- [contracts/SafEth/derivatives/SfrxEth.sol#L6](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L6)
- [contracts/SafEth/derivatives/SfrxEth.sol#L7](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L7)
- [contracts/SafEth/derivatives/SfrxEth.sol#L8](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L8)
- [contracts/SafEth/derivatives/SfrxEth.sol#L9](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L9)

```solidity
File: contracts/SafEth/SafEth.sol

4: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

5: import "../interfaces/IWETH.sol";

6: import "../interfaces/uniswap/ISwapRouter.sol";

7: import "../interfaces/lido/IWStETH.sol";

8: import "../interfaces/lido/IstETH.sol";

9: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

10: import "./SafEthStorage.sol";

11: import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

```
- [contracts/SafEth/SafEth.sol#L4](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/safeth/SafEth.sol#L4)
- [contracts/SafEth/SafEth.sol#L5](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/safeth/SafEth.sol#L5)
- [contracts/SafEth/SafEth.sol#L6](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/safeth/SafEth.sol#L6)
- [contracts/SafEth/SafEth.sol#L7](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/safeth/SafEth.sol#L7)
- [contracts/SafEth/SafEth.sol#L8](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/safeth/SafEth.sol#L8)
- [contracts/SafEth/SafEth.sol#L9](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/safeth/SafEth.sol#L9)
- [contracts/SafEth/SafEth.sol#L10](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/safeth/SafEth.sol#L10)
- [contracts/SafEth/SafEth.sol#L11](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/safeth/SafEth.sol#L11)

&nbsp;
## [N-05] Implementing `renounceOwnership` is dangerous

Typically, the contract's owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The OpenZeppelin's Ownable used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design.

Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

It is recommended to either reimplement the function to disable it, or to clearly specify if it is part of the contract design.


Instances: 4
```solidity
File: contracts/SafEth/derivatives/Reth.sol

19: contract Reth is IDerivative, Initializable, OwnableUpgradeable {

```
- [contracts/SafEth/derivatives/Reth.sol#L19](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L19)

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

12: contract WstEth is IDerivative, Initializable, OwnableUpgradeable {

```
- [contracts/SafEth/derivatives/WstEth.sol#L12](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L12)

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

13: contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {

```
- [contracts/SafEth/derivatives/SfrxEth.sol#L13](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L13)

```solidity
File: contracts/SafEth/SafEth.sol

15: contract SafEth is

```
- [contracts/SafEth/SafEth.sol#L15](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L15)

&nbsp;
## [N-06] Remove unused imports

Instances: 6
```solidity
File: contracts/SafEth/derivatives/Reth.sol

5: import "../../interfaces/frax/IsFrxEth.sol";

```
- [contracts/SafEth/derivatives/Reth.sol#L5](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L5)

```solidity
File: contracts/SafEth/SafEth.sol

4: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

5: import "../interfaces/IWETH.sol";

6: import "../interfaces/uniswap/ISwapRouter.sol";

7: import "../interfaces/lido/IWStETH.sol";

8: import "../interfaces/lido/IstETH.sol";

```
- [contracts/SafEth/SafEth.sol#L4](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L4)
- [contracts/SafEth/SafEth.sol#L5](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L5)
- [contracts/SafEth/SafEth.sol#L6](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L6)
- [contracts/SafEth/SafEth.sol#L7](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L7)
- [contracts/SafEth/SafEth.sol#L8](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L8)

&nbsp;
## [N-07] Missing `address(0)` checks in constructor/initialize

Failing to check for invalid parameters on deployment may result in an erroneous input and require an expensive redeployment.

Instances: 3
```solidity
File: contracts/SafEth/derivatives/Reth.sol

42:     function initialize(address _owner) external initializer {

```
- [contracts/SafEth/derivatives/Reth.sol#L42](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L42)

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

33:     function initialize(address _owner) external initializer {

```
- [contracts/SafEth/derivatives/WstEth.sol#L33](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L33)

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

36:     function initialize(address _owner) external initializer {

```
- [contracts/SafEth/derivatives/SfrxEth.sol#L36](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L36)

&nbsp;
## [N-08] Use built in constants over magic numbers

Improves code readability and reduces margin for error e.g. `200 ether` instead of `200 * 10 ** 18`.

Instances: 2
```solidity
File: contracts/SafEth/SafEth.sol

54:        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum

55:        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
```
- [contracts/SafEth/SafEth.sol#L54](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L54)
- [contracts/SafEth/SafEth.sol#L55](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L55)

```solidity
File: contracts/SafEth/derivatives/Reth.sol

44:        maxSlippage = (1 * 10 ** 16); // 1%
```
- [contracts/SafEth/derivatives/Reth.sol#L44](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L44)

&nbsp;
## [N-09] Typo in comments

Comment for `SafEth.adjustWeight` is misleading, as it claims that a weight is being added rather than adjusted.

Instances: 1
```solidity
File: contracts/SafEth/derivatives/Reth.sol

54:        @notice - Adds new derivative to the index fund
```

- [contracts/SafEth/SafEth.sol#L158](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L158)
