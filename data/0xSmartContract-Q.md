In terms of Quality Assurance;

Strengths of the Project;
-- The code has a structure written in a modern way, within the framework of generally accepted Soliditiy principles.
-- Details such as the fact that the `initialize()` function is declared externally, it has quality and short NatSpec comments, and there are almost no typos, show us the importance given to the code.

Development Areas of the Project;
-- The project uses the standard one-step owner model. 2-step ownership transfer is used in the most important projects in the new period, so 2-step ownership should be transferred [L-01]
-- The use of 'emit' in general terms, specifying new and old values in emit parameters is missing, there is a development area here, also sending 'emit' after ether transmission should be re-evaluated in terms of security, [L-03] [L-08]
-- `OnlyOwner` strength is not looked after as it is out of scope, but this is the weakest context of the project, here if timelock - multisign architectures are used together it will be the most robust structure and will increase trust in the project
-- How is the proxy architecture of the project, which pattern is used, how to upgrade parts are missing, adding these strengthens the control and security structure [N-26]


## Summary
### Low Risk Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-01]|Use `Ownable2StepUpgradeable` instead of ` OwnableUpgradeable ` contract| 4 |
|[L-02]|There may be a problem when changing the `maxSlippage` value because there is no range | 3 |
|[L-03]|Should declare to `emit` before the send ether in codebase for re-entrancy pattern | 1 |
|[L-04]|Omissions in Events|3 |
|[L-05]|Avoid using hardcode address in codebase| 11 |
|[L-06]|Lack of control to assign 0 values in the value assignments of critical state variables in the initialize| 3 |
|[L-07]|Use the latest updated version of OpenZeppelin dependencies| 1 |
|[L-08]|Add parameter to Event-Emit for critical function| 3 |
|[L-09]|Insufficient coverage| 1 |
|[L-10]|Prevent division by `0`| 1 |
| [N-11] |Add a timelock to critical functions|1|
| [N-12] |Include return parameters in NatSpec comments|All Contracts|
| [N-13] |Tokens accidentally sent to the contract cannot be recovered| 1 |
| [N-14] |For modern and more readable code; update import usages| 31 |
| [N-15] |Use a more recent version of Solidity|4 |
| [N-16] |Floating pragma | 4 |
| [N-17] |Project Upgrade and Stop Scenario should be|  |
| [N-18] |Use SMTChecker| |
| [N-19] |There is no need to cast a variable that is already an address, such as address(x)| 1 |
| [N-20] |Lines are too long|   |
| [N-21] |Use `uint256` instead `uint`|16 |
| [N-22] |`Function writing` that does not comply with the `Solidity Style Guide` | All Contracts |
| [N-23] |Avoid _shadowing_ `inherited state variables`| 1 |
| [N-24] |Constants on the left are better | 5 |
| [N-25] |Use a single file for all system-wide constants| 11 |
| [N-26] |Include proxy contracts to Audit| 1 |
| [N-27] |Missing Event for  initialize | 4 |
| [N-28] |Use Fuzzing Test for math code bases  |  |

Total 28 issues


### [L-01] Use `Ownable2StepUpgradeable` instead of ` OwnableUpgradeable ` contract

**Context:**
```solidity
4 results

contracts/SafEth/SafEth.sol:
   9: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
  15: contract SafEth is Initializable, ERC20Upgradeable, OwnableUpgradeable, SafEthStorage

contracts/SafEth/derivatives/Reth.sol:
  13: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
  19: contract Reth is IDerivative, Initializable, OwnableUpgradeable {

contracts/SafEth/derivatives/SfrxEth.sol:
   6: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
  13: contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {

contracts/SafEth/derivatives/WstEth.sol:
   5: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
  12: contract WstEth is IDerivative, Initializable, OwnableUpgradeable {

```


**Description:**
```transferOwnership``` function is used to change Ownership from ```OwnableUpgradeable.sol```.

There is another Openzeppelin Ownable contract (Ownable2StepUpgradeable.sol) has  ` transferOwnership` function ,  use it is more secure due to 2-stage ownership transfer.

[Ownable2StepUpgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol)

### [L-02] There may be a problem when changing the `maxSlippage` value because there is no range


**Context:**
```solidity

3 results - 3 files

contracts/SafEth/derivatives/Reth.sol:
  57      */
  58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {
  59:         maxSlippage = _slippage;
  60:     }
  61  

contracts/SafEth/derivatives/SfrxEth.sol:
  50      */
  51:     function setMaxSlippage(uint256 _slippage) external onlyOwner {
  52:         maxSlippage = _slippage;
  53:     }
  54  

contracts/SafEth/derivatives/WstEth.sol:
  47      */
  48:     function setMaxSlippage(uint256 _slippage) external onlyOwner {
  49:         maxSlippage = _slippage;
  50:     }


```

**Description:**
The onlyOwner has the authority to change the `maxSlippage` value, but there are no upper and lower controls on the value during the change, so there is a risk of setting the value so high that it raises questions for users.

**Recommendation**

Specify to value range `maxSlippage` for variable change in `setMaxSlippage` functions


### [L-03] Should declare to `emit` before the send ether in codebase for re-entrancy pattern



```diff

1 result - 1 file

contracts/SafEth/SafEth.sol:
  103      */
  104:     function unstake(uint256 _safEthAmount) external {
  105:         require(pauseUnstaking == false, "unstaking is paused");
  106:         uint256 safEthTotalSupply = totalSupply();
  107:         uint256 ethAmountBefore = address(this).balance;
  108: 
  109:         for (uint256 i = 0; i < derivativeCount; i++) {
  110:             // withdraw a percentage of each asset based on the amount of safETH
  111:             uint256 derivativeAmount = (derivatives[i].balance() *
  112:                 _safEthAmount) / safEthTotalSupply;
  113:             if (derivativeAmount == 0) continue; // if derivative empty ignore
  114:             derivatives[i].withdraw(derivativeAmount);
  115:         }
  116:         _burn(msg.sender, _safEthAmount);
  117:         uint256 ethAmountAfter = address(this).balance;
  118:         uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;
+ 119:        emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
  120:         (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
  121:             ""
  122:         );
  123:         require(sent, "Failed to send Ether");
- 124:         emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
  125:     }


```


### [L-04] Omissions in Events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

```solidity 

3 result

contracts/SafEth/SafEth.sol:
  197      */
  198:     function setMaxSlippage(
  199:         uint _derivativeIndex,
  200:         uint _slippage ) external onlyOwner {
  202:         derivatives[_derivativeIndex].setMaxSlippage(_slippage);
  203:         emit SetMaxSlippage(_derivativeIndex, _slippage);
  204:     }
  205: 
  210:     function setMinAmount(uint256 _minAmount) external onlyOwner {
  211:         minAmount = _minAmount;
  212:         emit ChangeMinAmount(minAmount);
  213:     }
  214: 
  219:     function setMaxAmount(uint256 _maxAmount) external onlyOwner {
  220:         maxAmount = _maxAmount;
  221:         emit ChangeMaxAmount(maxAmount);
  222:     }


```

### [L-05] Avoid using hardcode address in codebase

Hardcoding the Router and Factory addresses may cause me to not be able to change this address in the future and not be able to switch to new versions.

Let's not forget that there have been transitions such as uniswap v1 - uniswap v2 - uniswap v3, pancakeswap v1 - pancakeswap v2, and the router-factory addresses have changed.

```solidity
11 results - 3 files

contracts/SafEth/derivatives/Reth.sol:
  19  contract Reth is IDerivative, Initializable, OwnableUpgradeable {
  20:     address public constant ROCKET_STORAGE_ADDRESS =
  21          0x1d8f8f00cfa6758d7bE78336684788Fb0ee0Fa46;
  22:     address public constant W_ETH_ADDRESS =
  23          0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;
  24:     address public constant UNISWAP_ROUTER =
  25          0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45;
  26:     address public constant UNI_V3_FACTORY =
  27          0x1F98431c8aD98523631AE4a59f267346ea31F984;

contracts/SafEth/derivatives/SfrxEth.sol:
  13  contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {
  14:     address public constant SFRX_ETH_ADDRESS =
  15          0xac3E018457B222d93114458476f3E3416Abbe38F;
  16:     address public constant FRX_ETH_ADDRESS =
  17          0x5E8422345238F34275888049021821E8E08CAa1f;
  18:     address public constant FRX_ETH_CRV_POOL_ADDRESS =
  19          0xa1F8A6807c402E4A15ef4EBa36528A3FED24E577;
  20:     address public constant FRX_ETH_MINTER_ADDRESS =
  21          0xbAFA44EFE7901E04E39Dad13167D089C559c1138;

contracts/SafEth/derivatives/WstEth.sol:
  12  contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
  13:     address public constant WST_ETH =
  14          0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0;
  15:     address public constant LIDO_CRV_POOL =
  16          0xDC24316b9AE028F1497c275EB9192a3Ea0f67022;
  17:     address public constant STETH_TOKEN =
  18          0xae7ab96520DE3A18E5e111B5EaAb095312D7fE84;


```

Recommendation:

Use changeable architecture for router and factory addresses


### [L-06] Lack of control to assign 0 values in the value assignments of critical state variables in the initialize


`_owner` address variable is critical value. So should be check 0 value in initialize, if the _owner is 0 by mistake, even for a while, it will cause a loss to the project.


```solidity
3 results - 3 files

contracts/SafEth/derivatives/Reth.sol:
  41      */
  42:     function initialize(address _owner) external initializer {
  43:         _transferOwnership(_owner);
  44:         maxSlippage = (1 * 10 ** 16); // 1%
  45:     }
  46  

contracts/SafEth/derivatives/SfrxEth.sol:
  35      */
  36:     function initialize(address _owner) external initializer {
  37:         _transferOwnership(_owner);
  38:         maxSlippage = (1 * 10 ** 16); // 1%
  39:     }
  40  

contracts/SafEth/derivatives/WstEth.sol:
  32      */
  33:     function initialize(address _owner) external initializer {
  34:         _transferOwnership(_owner);
  35:         maxSlippage = (1 * 10 ** 16); // 1%
  36:     }
```


### [L-07] Use the latest updated version of OpenZeppelin dependencies

```js
package.json:
  76    },
  77:   "dependencies": {
  82:     "@openzeppelin/contracts": "^4.8.0",
  83:     "@openzeppelin/contracts-upgradeable": "^4.8.1",

```

### Proof Of Concept

https://github.com/OpenZeppelin/openzeppelin-contracts/releases/

https://security.snyk.io/vuln/SNYK-JS-OPENZEPPELINCONTRACTSUPGRADEABLE-3339524?utm_medium=Partner&utm_source=RedHat&utm_campaign=Code-Ready-Analytics-2020&utm_content=vuln%2FSNYK-JS-OPENZEPPELINCONTRACTSUPGRADEABLE-3339524


### Recommended Mitigation Steps
Upgrade `OpenZeppelin` to version 4.8.2


### [L-08] Add parameter to Event-Emit for critical function

Add to parameter  for front-end website or client app , they can has that something has happened on the blockchain.
Especially for important changes
```solidity 

3 result

contracts/SafEth/derivatives/Reth.sol:
  57      */
  58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {
  59:         maxSlippage = _slippage;
  60:     }
  61  

contracts/SafEth/derivatives/SfrxEth.sol:
  50      */
  51:     function setMaxSlippage(uint256 _slippage) external onlyOwner {
  52:         maxSlippage = _slippage;
  53:     }
  54  

contracts/SafEth/derivatives/WstEth.sol:
  47      */
  48:     function setMaxSlippage(uint256 _slippage) external onlyOwner {
  49:         maxSlippage = _slippage;
  50:     }



```

### [L-09] Insufficient coverage

**Description:**
The test coverage rate of the project is ~80%. Testing all functions is best practice in terms of security criteria.
Due to its capacity, test coverage is expected to be 100%.


```js
1 result - 1 file

138: - What is the overall line coverage percentage provided by your tests?:  92

```


### [L-10] Prevent division by `0`

On several locations in the code precautions are not being taken for not dividing by 0, this will revert the code.
These functions can be calledd with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero.

`totalWeight` value should be check for 0 value

```diff

contracts/SafEth/SafEthStorage.sol:
  19:     uint256 public totalWeight; // total weight of all derivatives (used to calculate percentage of derivative)


contracts/SafEth/SafEth.sol:

84:             uint256 ethAmount = (msg.value * weight) / totalWeight;

```


### [N-11] Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users.


Consider adding a timelock to:


```solidity

1 result - 1 file

contracts/SafEth/SafEth.sol:
  209      */
  210:     function setMinAmount(uint256 _minAmount) external onlyOwner {
  211:         minAmount = _minAmount;
  212:         emit ChangeMinAmount(minAmount);
  213:     }

```


### [N-12] Include return parameters in NatSpec comments

**Context:**
All Contracts

**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Recommendation:**
Include return parameters in NatSpec comments

_Recommendation  Code Style: (from Uniswap3)_
```js
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

### [N-13] Tokens accidentally sent to the contract cannot be recovered

stETH , ETH and sfrx ETH tokens may be sent to the stake contract by mistake
contracts/SafEth/SafEth.sol:

It can't be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

### Recommended Mitigation Steps

Add this code:

```solidity
 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

```


### [N-14] For modern and more readable code; update import usages

**Context:**

```solidity

31 results - 4 files

contracts/SafEth/SafEth.sol:
   3  
   4: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
   5: import "../interfaces/IWETH.sol";
   6: import "../interfaces/uniswap/ISwapRouter.sol";
   7: import "../interfaces/lido/IWStETH.sol";
   8: import "../interfaces/lido/IstETH.sol";
   9: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
  10: import "./SafEthStorage.sol";
  11: import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
  12  

contracts/SafEth/derivatives/Reth.sol:
   3  
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
  16  

contracts/SafEth/derivatives/SfrxEth.sol:
  3  
  4: import "../../interfaces/IDerivative.sol";
  5: import "../../interfaces/frax/IsFrxEth.sol";
  6: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
  7: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
  8: import "../../interfaces/curve/IFrxEthEthPool.sol";
  9: import "../../interfaces/frax/IFrxETHMinter.sol";
  10  

contracts/SafEth/derivatives/WstEth.sol:
  3  
  4: import "../../interfaces/IDerivative.sol";
  5: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
  6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
  7: import "../../interfaces/curve/IStEthEthPool.sol";
  8: import "../../interfaces/lido/IWStETH.sol";

```


**Description:**
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it. 
This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces allow us to apply this rule better.

**Recommendation:**
`import {contract1 , contract2} from "filename.sol";`

A good example from the ArtGobblers project;
```js
import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";
```
### [N-15] Use a more recent version of Solidity

**Context:**
```solidity
4 results - 4 files

contracts/SafEth/SafEth.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.13;
  3  

contracts/SafEth/derivatives/Reth.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.13;
  3  

contracts/SafEth/derivatives/SfrxEth.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.13;
  3  

contracts/SafEth/derivatives/WstEth.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.13;

```
**Description:**
For security, it is best practice to use the latest Solidity version.
For the security fix list in the versions;
https://github.com/ethereum/solidity/blob/develop/Changelog.md


**Recommendation:**
Old version of Solidity is used `(0.8.8)`, newer version can be used `(0.8.17)` 

### [N-16] Floating pragma

**Description:**
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

Floating Pragma List: 
```solidity
4 results - 4 files

contracts/SafEth/SafEth.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.13;
  3  

contracts/SafEth/derivatives/Reth.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.13;
  3  

contracts/SafEth/derivatives/SfrxEth.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.13;
  3  

contracts/SafEth/derivatives/WstEth.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.13;


```


**Recommendation:**
Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

### [N-17] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol


### [N-18] Use SMTChecker

The *highest* tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## [N-19] There is no need to cast a variable that is already an address, such as address(x)

```diff

1 result - 1 file

contracts/SafEth/SafEth.sol:
  103      */
  104:     function unstake(uint256 _safEthAmount) external {
  105:         require(pauseUnstaking == false, "unstaking is paused");
  106:         uint256 safEthTotalSupply = totalSupply();
  107:         uint256 ethAmountBefore = address(this).balance;
  108: 
  109:         for (uint256 i = 0; i < derivativeCount; i++) {
  110:             // withdraw a percentage of each asset based on the amount of safETH
  111:             uint256 derivativeAmount = (derivatives[i].balance() *
  112:                 _safEthAmount) / safEthTotalSupply;
  113:             if (derivativeAmount == 0) continue; // if derivative empty ignore
  114:             derivatives[i].withdraw(derivativeAmount);
  115:         }
  116:         _burn(msg.sender, _safEthAmount);
  117:         uint256 ethAmountAfter = address(this).balance;
  118:         uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;
  119:         // solhint-disable-next-line
- 120:         (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
+ 	 (bool sent, ) = msg.sender.call{value: ethAmountToWithdraw}(
  121:             ""
  122:         );
  123:         require(sent, "Failed to send Ether");
  124:         emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
  125:     }

```
**Description:**
There is no need to cast a variable that is already an address, such as address(....) msg.sender also address.

**Recommendation:**
Use directly variable.

### [N-20] Lines are too long

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases.
Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

There are many examples, some of which are listed below;

[Reth.sol#L142](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L142)

### [N-21] Use `uint256` instead `uint`

Project use uint and uint256
 
Number of uses:
uint  = 16 results
uint256 = 73 results

Some developers prefer to use `uint256` because it is consistent with other uint data types, which also specify their size, and also because making the size of the data explicit reminds the developer and the reader how much data they've got to play with, which may help prevent or detect bugs.

For example if doing ```bytes4(keccak('transfer(address, uint)’))```, you'll get a different method sig ID than ```bytes4(keccak('transfer(address, uint256)’))``` and smart contracts will only understand the latter when comparing method sig IDs


### [N-22] `Function writing` that does not comply with the `Solidity Style Guide`

**Context:**
All Contracts

**Description:**
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

 constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last


## [N-23] Avoid _shadowing_ `inherited state variables`

**Context:**
```solidity

node_modules/@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol:
   98       */
   99:     function totalSupply() public view virtual override returns (uint256) {
  100:         return _totalSupply;


contracts/SafEth/SafEth.sol:
  73:         uint256 totalSupply = totalSupply();
  
  77:         else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

```
`totalSupply` is shadowed, 

The local variable name ` totalSupply ` in the `SafEth.sol` hase the same name and create shadowing

**Recommendation:**
Avoid using variables with the same name, including inherited in the same contract, if used, it must be specified in the NatSpec comments.


### [N-24] Constants on the left are better

If you use the constant first you support structures that veil programming errors. And one should only produce code either to add functionality, fix an programming error or trying to support structures to avoid programming errors (like design patterns). 

https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html

```solidity

5 results - 1 file

contracts/SafEth/SafEth.sol:
   74          uint256 preDepositPrice; // Price of safETH in regards to ETH
   75:         if (totalSupply == 0)

   83:             if (weight == 0) continue;
   84              uint256 ethAmount = (msg.value * weight) / totalWeight;

  113:             if (derivativeAmount == 0) continue; // if derivative empty ignore
  114              derivatives[i].withdraw(derivativeAmount);

  143          for (uint i = 0; i < derivativeCount; i++) {
  144:             if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
  145              uint256 ethAmount = (ethAmountToRebalance * weights[i]) /

```


### [N-25] Use a single file for all system-wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values)

  This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

```solidity

11 results - 3 files

contracts/SafEth/derivatives/Reth.sol:
  19  contract Reth is IDerivative, Initializable, OwnableUpgradeable {
  20:     address public constant ROCKET_STORAGE_ADDRESS =
  21          0x1d8f8f00cfa6758d7bE78336684788Fb0ee0Fa46;
  22:     address public constant W_ETH_ADDRESS =
  23          0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;
  24:     address public constant UNISWAP_ROUTER =
  25          0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45;
  26:     address public constant UNI_V3_FACTORY =
  27          0x1F98431c8aD98523631AE4a59f267346ea31F984;

contracts/SafEth/derivatives/SfrxEth.sol:
  13  contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {
  14:     address public constant SFRX_ETH_ADDRESS =
  15          0xac3E018457B222d93114458476f3E3416Abbe38F;
  16:     address public constant FRX_ETH_ADDRESS =
  17          0x5E8422345238F34275888049021821E8E08CAa1f;
  18:     address public constant FRX_ETH_CRV_POOL_ADDRESS =
  19          0xa1F8A6807c402E4A15ef4EBa36528A3FED24E577;
  20:     address public constant FRX_ETH_MINTER_ADDRESS =
  21          0xbAFA44EFE7901E04E39Dad13167D089C559c1138;

contracts/SafEth/derivatives/WstEth.sol:
  12  contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
  13:     address public constant WST_ETH =
  14          0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0;
  15:     address public constant LIDO_CRV_POOL =
  16          0xDC24316b9AE028F1497c275EB9192a3Ea0f67022;
  17:     address public constant STETH_TOKEN =
  18          0xae7ab96520DE3A18E5e111B5EaAb095312D7fE84;


```
## [N-26] Include proxy contracts to Audit

The Proxy contract could not be seen in the checklist because it is an upgradable contract, only the implementation contracts are visible, I recommend including the Proxy contract in the audit for integrity in the audit.


### [N-27] Missing Event for  initialize

**Context:**
```solidity

4 results - 4 files

contracts/SafEth/SafEth.sol:
  48:     function initialize(

contracts/SafEth/derivatives/Reth.sol:
  42:     function initialize(address _owner) external initializer {

contracts/SafEth/derivatives/SfrxEth.sol:
  36:     function initialize(address _owner) external initializer {

contracts/SafEth/derivatives/WstEth.sol:
  32      */
  33:     function initialize(address _owner) external initializer {

```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
Issuing event-emit during initialization is a detail that many projects skip

**Recommendation:**
Add Event-Emit in initialize
### [N-28]  Use Fuzzing Test for math code bases 


I recommend fuzzing testing in math code structures,

**Recommendation:**

Use should fuzzing test like Echidna.


Fuzzing is not easy, the tools are rough, and the math is hard, but it is worth it. Fuzzing gives me a level of confidence in my smart contracts that I didn’t have before. Relying just on unit testing anymore and poking around in a testnet seems reckless now.



https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05

