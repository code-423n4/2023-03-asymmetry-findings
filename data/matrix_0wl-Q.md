# Summary

## Low Risk and Non-Critical Issues

## Non-Critical Issues

|       | Issue                                                                                               |
| ----- | :-------------------------------------------------------------------------------------------------- |
| NC-1  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                                                |
| NC-2  | BE EXPLICIT DECLARING TYPES                                                                         |
| NC-3  | BOOLEAN EQUALITY                                                                                    |
| NC-4  | USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)                                                |
| NC-5  | GENERATE PERFECT CODE HEADERS EVERY TIME                                                            |
| NC-6  | USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS                                                     |
| NC-7  | FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES                                             |
| NC-8  | LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS                                        |
| NC-9  | LACK OF CHECKS SUPPORTSINTERFACE                                                                    |
| NC-10 | MISSING EVENT FOR CRITICAL PARAMETER CHANGE                                                         |
| NC-11 | MISSING FEE PARAMETER VALIDATION                                                                    |
| NC-12 | NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS                                                   |
| NC-13 | INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS                                                       |
| NC-14 | NO NEED TO RE-INHERIT                                                                               |
| NC-15 | NO SAME VALUE INPUT CONTROL                                                                         |
| NC-16 | ADD PARAMETER TO EVENT-EMIT                                                                         |
| NC-17 | FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE                                 |
| NC-18 | BETTER TO HAVE CONFIG FACET                                                                         |
| NC-19 | DON’T USE PERIODS WITH FRAGMENTS                                                                    |
| NC-20 | USE A MORE RECENT VERSION OF SOLIDITY                                                               |
| NC-21 | UNUSED IMPORTS                                                                                      |
| NC-22 | FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS                             |
| NC-23 | LINES ARE TOO LONG                                                                                  |
| NC-24 | IMPLEMENT SOME TYPE OF VERSION COUNTER THAT WILL BE INCREMENTED AUTOMATICALLY FOR CONTRACT UPGRADES |

### [NC-1] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

202:     function setMaxSlippage(

214:     function setMinAmount(uint256 _minAmount) external onlyOwner {

223:     function setMaxAmount(uint256 _maxAmount) external onlyOwner {

232:     function setPauseStaking(bool _pause) external onlyOwner {

241:     function setPauseUnstaking(bool _pause) external onlyOwner {

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

51:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

48:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

```

### [NC-2] BE EXPLICIT DECLARING TYPES

#### Description:

Instead of uint use uint256

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

26:     event Staked(address indexed recipient, uint ethIn, uint safEthOut);

27:     event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);

28:     event WeightChange(uint indexed index, uint weight);

31:         uint weight,

32:         uint index

71:         for (uint i = 0; i < derivativeCount; i++)

84:         for (uint i = 0; i < derivativeCount; i++) {

92:             uint derivativeReceivedEthValue = (derivative.ethPerDerivative(

140:         for (uint i = 0; i < derivativeCount; i++) {

147:         for (uint i = 0; i < derivativeCount; i++) {

203:         uint _derivativeIndex,

204:         uint _slippage

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

171:             uint rethPerEth = (10 ** 36) / poolPrice();

241:         return (sqrtPriceX96 * (uint(sqrtPriceX96)) * (1e18)) >> (96 * 2);

```

### [NC-3] BOOLEAN EQUALITY

#### Description:

Boolean variables can be checked within conditionals directly without the use of equality operators to true/false.

Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#boolean-equality

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

64:         require(pauseStaking == false, "staking is paused");

109:         require(pauseUnstaking == false, "unstaking is paused");

```

### [NC-4] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)

#### Description:

Rather than using `abi.encodePacked` for appending bytes, since version 0.8.4, `bytes.concat()` is enabled.

Since version 0.8.4 for appending bytes, `bytes.concat()` can be used instead of `abi.encodePacked(,)`.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/derivatives/Reth.sol

70:                     abi.encodePacked("contract.address", "rocketTokenRETH")

125:                     abi.encodePacked("contract.address", "rocketDepositPool")

162:                     abi.encodePacked("contract.address", "rocketDepositPool")

191:                         abi.encodePacked("contract.address", "rocketTokenRETH")

233:                     abi.encodePacked("contract.address", "rocketTokenRETH")

```

### [NC-5] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-6] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS

#### Description:

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes.

Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/derivatives/Reth.sol

20:     address public constant ROCKET_STORAGE_ADDRESS =

22:     address public constant W_ETH_ADDRESS =

24:     address public constant UNISWAP_ROUTER =

26:     address public constant UNI_V3_FACTORY =

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

14:     address public constant SFRX_ETH_ADDRESS =

16:     address public constant FRX_ETH_ADDRESS =

18:     address public constant FRX_ETH_CRV_POOL_ADDRESS =

20:     address public constant FRX_ETH_MINTER_ADDRESS =

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

13:     address public constant WST_ETH =

15:     address public constant LIDO_CRV_POOL =

17:     address public constant STETH_TOKEN =

```

### [NC-7] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES

#### Description:

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

4: import "../interfaces/IWETH.sol";

6: import "../interfaces/uniswap/ISwapRouter.sol";

7: import "../interfaces/lido/IWStETH.sol";

8: import "../interfaces/lido/IstETH.sol";

10: import "./SafEthStorage.sol";

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

4: import "../../interfaces/IDerivative.sol";

5: import "../../interfaces/frax/IsFrxEth.sol";

7: import "../../interfaces/rocketpool/RocketStorageInterface.sol";

9: import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";

10: import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";

11: import "../../interfaces/IWETH.sol";

12: import "../../interfaces/uniswap/ISwapRouter.sol";

14: import "../../interfaces/uniswap/IUniswapV3Factory.sol";

15: import "../../interfaces/uniswap/IUniswapV3Pool.sol";

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

4: import "../../interfaces/IDerivative.sol";

5: import "../../interfaces/frax/IsFrxEth.sol";

6: import "../../interfaces/IDerivative.sol";

8: import "../../interfaces/curve/IFrxEthEthPool.sol";

9: import "../../interfaces/frax/IFrxETHMinter.sol";

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

4: import "../../interfaces/IDerivative.sol";

7: import "../../interfaces/curve/IStEthEthPool.sol";

8: import "../../interfaces/lido/IWStETH.sol";

```

#### Recommended Mitigation Steps:

`import {contract1 , contract2} from "filename.sol";` OR Use specific imports syntax per solidity docs recommendation.

### [NC-8] LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS

#### Description:

To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the initialize() functions:

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

48:     function initialize(

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

42:     function initialize(address _owner) external initializer {

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

36:     function initialize(address _owner) external initializer {

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

33:     function initialize(address _owner) external initializer {

```

### [NC-9] LACK OF CHECKS SUPPORTSINTERFACE

#### Description:

The EIP-165 standard helps detect that a smart contract implements the expected logic, prevents human error when configuring smart contract bindings, so it is recommended to check that the received argument is a contract and supports the expected interface.

Reference: https://eips.ethereum.org/EIPS/eip-165

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

```

### [NC-10] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

#### Description:

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

https://github.com/crytic/slither/wiki/Detector-Documentation#missing-events-access-control

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/derivatives/Reth.sol

58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

51:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

48:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

```

### [NC-11] MISSING FEE PARAMETER VALIDATION

#### Description:

Some fee parameters of functions are not checked for invalid values. Validate the parameters.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/derivatives/Reth.sol

86:         uint24 _poolFee,

```

### [NC-12] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS

#### Description:

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.

In complex projects the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html.

#### Recommended Mitigation Steps:

NatSpec comments should be increased in contracts

### [NC-13] INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS

#### Description:

If Return parameters are declared, you must prefix them with ”/// @return”.

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

Include return parameters in NatSpec comments

```solidity
File: contracts/SafEth/derivatives/Reth.sol

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

```

### [NC-14] NO NEED TO RE-INHERIT

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

15: contract SafEth is

import "../../interfaces/IDerivative.sol";
import "../../interfaces/frax/IsFrxEth.sol";
import "../../interfaces/curve/IFrxEthEthPool.sol";
import "../../interfaces/frax/IFrxETHMinter.sol";

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

import "../../interfaces/frax/IsFrxEth.sol";
import "../../interfaces/rocketpool/RocketStorageInterface.sol";
import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
import "../../interfaces/IWETH.sol";
import "../../interfaces/uniswap/ISwapRouter.sol";
import "../../interfaces/uniswap/IUniswapV3Factory.sol";
import "../../interfaces/uniswap/IUniswapV3Pool.sol";

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

import "../../interfaces/frax/IsFrxEth.sol";
import "../../interfaces/IDerivative.sol";
import "../../interfaces/frax/IsFrxEth.sol";
import "../../interfaces/curve/IFrxEthEthPool.sol";
import "../../interfaces/frax/IFrxETHMinter.sol";

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

import "../../interfaces/curve/IStEthEthPool.sol";
import "../../interfaces/lido/IWStETH.sol";

```

### [NC-15] NO SAME VALUE INPUT CONTROL

#### Recommended Mitigation Steps:

Add code like this; `if (oracle == _oracle revert ADDRESS_SAME();`

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

215:         minAmount = _minAmount;

224:         maxAmount = _maxAmount;

```

### [NC-16] ADD PARAMETER TO EVENT-EMIT

#### Description:

Some event-emit description hasn’t parameter. Add to parameter for front-end website or client app , they can has that something has happened on the blockchain.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

154:         emit Rebalanced();

216:         emit ChangeMinAmount(minAmount);

225:         emit ChangeMaxAmount(maxAmount);

234:         emit StakingPaused(pauseStaking);

243:         emit UnstakingPaused(pauseUnstaking);

```

### [NC-17] FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE

#### Description:

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private, within a grouping, place the view and pure functions last

### [NC-18] BETTER TO HAVE CONFIG FACET

Better to have config facet, in case some update is needed in the config.sol. Therefore, it is not necessary to redeploy the facets that imported config.

### [NC-19] DON’T USE PERIODS WITH FRAGMENTS

Throughout the files, most of the comments have fragments that end without periods. It should be consistent. The periods should be removed.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

163:        @param _weight - new weight for this derivative.
180:        @param _weight - new weight for this derivative.
```

### [NC-20] USE A MORE RECENT VERSION OF SOLIDITY

#### Context:

All contracts

#### Description:

For security, it is best practice to use the latest Solidity version.

For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

2: pragma solidity ^0.8.13;

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

2: pragma solidity ^0.8.13;

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

2: pragma solidity ^0.8.13;

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

2: pragma solidity ^0.8.13;

```

### [NC-21] UNUSED IMPORTS

```solidity
File: contracts/SafEth/SafEth.sol

15: contract SafEth is

import "../../interfaces/IDerivative.sol";
import "../../interfaces/frax/IsFrxEth.sol";
import "../../interfaces/curve/IFrxEthEthPool.sol";
import "../../interfaces/frax/IFrxETHMinter.sol";

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

import "../../interfaces/frax/IsFrxEth.sol";
import "../../interfaces/rocketpool/RocketStorageInterface.sol";
import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
import "../../interfaces/IWETH.sol";
import "../../interfaces/uniswap/ISwapRouter.sol";
import "../../interfaces/uniswap/IUniswapV3Factory.sol";
import "../../interfaces/uniswap/IUniswapV3Pool.sol";

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

import "../../interfaces/frax/IsFrxEth.sol";
import "../../interfaces/IDerivative.sol";
import "../../interfaces/frax/IsFrxEth.sol";
import "../../interfaces/curve/IFrxEthEthPool.sol";
import "../../interfaces/frax/IFrxETHMinter.sol";

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

import "../../interfaces/curve/IStEthEthPool.sol";
import "../../interfaces/lido/IWStETH.sol";

```

### [NC-22] FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS

#### Description:

Solidity’s standard naming convention for internal and private functions and variables (apart from constants): the mixedCase format starting with an underscore (\_mixedCase starting with an underscore)

Solidity’s standard naming convention for internal and private constants variables: the snake_case format starting with an underscore (\_mixedCase starting with an underscore) and use ALL_CAPS for naming them.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/derivatives/Reth.sol


20:     address public constant ROCKET_STORAGE_ADDRESS =

22:     address public constant W_ETH_ADDRESS =

24:     address public constant UNISWAP_ROUTER =

26:     address public constant UNI_V3_FACTORY =

66:     function rethAddress() private view returns (address) {

120:     function poolCanDeposit(uint256 _amount) private view returns (bool) {

228:     function poolPrice() private view returns (uint256) {

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

14:     address public constant SFRX_ETH_ADDRESS =

16:     address public constant FRX_ETH_ADDRESS =

18:     address public constant FRX_ETH_CRV_POOL_ADDRESS =

20:     address public constant FRX_ETH_MINTER_ADDRESS =

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

13:     address public constant WST_ETH =

15:     address public constant LIDO_CRV_POOL =

17:     address public constant STETH_TOKEN =

```

### [NC-23] LINES ARE TOO LONG

#### Description:

Usually lines in source code are limited to 80 characters.

[Reference](https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length)

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

61:         @dev - Mints safEth in a redeemable value which equals to the correct percentage of the total staked value

83:         uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system

133:         @dev - Withdraws all derivative and re-deposit them to have the correct weights

134:         @dev - Depending on the balance of the derivative this could cause bad slippage

135:         @dev - If weights are updated then it will slowly change over time to the correct weight distribution

159:         @dev - Weights are only in regards to each other, total weight changes with this function

160:         @dev - If you want exact weights either do the math off chain or reset all existing derivates to the weights you want

162:         @param _derivativeIndex - index of the derivative you want to update the weight

179:         @param _contractAddress - Address of the derivative contract launched by AF

199:         @param _derivativeIndex - index of the derivative you want to update the slippage

220:         @notice - Owner only function that sets the maximum amount a user is allowed to stake

238:         @notice - Owner only function that enables/disables the unstake function

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

10: import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";

128:         RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(

142:         RocketDAOProtocolSettingsDepositInterface rocketDAOProtocolSettingsDeposit = RocketDAOProtocolSettingsDepositInterface(

154:         @dev - will either get rETH on exchange or deposit into contract depending on availability

166:         RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(

194:             RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(

208:         @dev - we need to pass amount so that it gets price from the same source that it buys or mints the rEth

```

### [NC-24] IMPLEMENT SOME TYPE OF VERSION COUNTER THAT WILL BE INCREMENTED AUTOMATICALLY FOR CONTRACT UPGRADES

#### Description:

I suggest implementing some kind of version counter that will be incremented automatically when you upgrade the contract.

#### Recommendation

```solidity
uint256 public authorizeUpgradeCounter;

 function upgradeTo(address _newImplementation) external ifAdmin {
        _setPendingImplementation(_newImplementation);
       authorizeUpgradeCounter+=1;
    }
```

## Low Issues

|      | Issue                                                               |
| ---- | :------------------------------------------------------------------ |
| L-1  | USE OWNABLE2STEPUPGRADEABLE INSTEAD OF OWNABLEUPGRADEABLE CONTRACT  |
| L-2  | INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY                      |
| L-3  | LACK OF INPUT VALIDATION                                            |
| L-4  | LOSS OF PRECISION DUE TO ROUNDING                                   |
| L-5  | UNIFY RETURN CRITERIA                                               |
| L-6  | OWNER CAN RENOUNCE OWNERSHIP                                        |
| L-7  | A SINGLE POINT OF FAILURE                                           |
| L-8  | ACCOUNT EXISTENCE CHECK FOR LOW-LEVEL CALLS                         |
| L-9  | UNUSED `RECEIVE()` FUNCTION WILL LOCK ETHER IN CONTRACT             |
| L-10 | USE `_SAFEMINT` INSTEAD OF `_MINT`                                  |
| L-11 | USE `SAFETRANSFEROWNERSHIP` INSTEAD OF `TRANSFEROWNERSHIP` FUNCTION |

### [L-1] USE OWNABLE2STEPUPGRADEABLE INSTEAD OF OWNABLEUPGRADEABLE CONTRACT

#### Description:

transferOwnership function is used to change Ownership from OwnableUpgradeable.sol.

There is another Openzeppelin Ownable contract (Ownable2StepUpgradeable.sol) has transferOwnership function, use it is more secure due to 2-stage ownership transfer.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

18:     OwnableUpgradeable,

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

19: contract Reth is IDerivative, Initializable, OwnableUpgradeable {

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

13: contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

12: contract WstEth is IDerivative, Initializable, OwnableUpgradeable {

```

#### Recommended Mitigation Steps:

We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.

### [L-2] INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY

#### Description:

`initialize()` function can be called anybody when the contract is not initialized.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

51:     ) external initializer {

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

42:     function initialize(address _owner) external initializer {

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

36:     function initialize(address _owner) external initializer {

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

33:     function initialize(address _owner) external initializer {

```

### [L-3] LACK OF INPUT VALIDATION

#### Description:

For defence-in-depth purpose, it is recommended to perform additional validation against the amount that the user is attempting to deposit, mint, withdraw and redeem to ensure that the submitted amount is valid.
https://code4rena.com/reports/2022-11-redactedcartel#l-11-lack-of-input-validation

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/derivatives/Reth.sol

107:     function withdraw(uint256 amount) external onlyOwner {

177:             uint256 amountSwapped = swapExactInputSingleHop(

```

### [L-4] LOSS OF PRECISION DUE TO ROUNDING

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

73:                 (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74:                        derivatives[i].balance()) /
75:                 10 ** 18;

88:             uint256 ethAmount = (msg.value * weight) / totalWeight;

92:         uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
93:                depositAmount
94:            ) * depositAmount) / 10 ** 18;

115:             uint256 derivativeAmount = (derivatives[i].balance() *
116:                 _safEthAmount) / safEthTotalSupply;

149:             uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
150:                totalWeight;

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

173:             uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
174:                 ((10 ** 18 - maxSlippage))) / 10 ** 18);

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

74:         uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
75:             (10 ** 18 - maxSlippage)) / 10 ** 18;

115:         return ((10 ** 18 * frxAmount) /
116:             IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

60:         uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;

```

### [L-5] UNIFY RETURN CRITERIA

#### Description:

In Reth.sol file sometimes the name of the return variable is not defined and sometimes is, unifying the way of writing the code makes the code more uniform and readable.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/derivatives/Reth.sol

50:     function name() public pure returns (string memory) {

66:     function rethAddress() private view returns (address) {

89:     ) private returns (uint256 amountOut) {

120:     function poolCanDeposit(uint256 _amount) private view returns (bool) {

156:     function deposit() external payable onlyOwner returns (uint256) {

211:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {

221:     function balance() public view returns (uint256) {

228:     function poolPrice() private view returns (uint256) {

```

### [L-6] OWNER CAN RENOUNCE OWNERSHIP

#### Description:

Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The Openzeppelin’s Ownable used in this project contract implements `renounceOwnership`. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

#### Recommended Mitigation Steps:

We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.`

### [L-7] A SINGLE POINT OF FAILURE

#### Description:

The owner role has a single point of failure and onlyOwner can use critical a few functions.

Owner is not behind a multisig and changes are not behind a timelock.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

138:     function rebalanceToWeights() external onlyOwner {

165:     function adjustWeight(
            uint256 _derivativeIndex,
            uint256 _weight
        ) external onlyOwner {

182:     function addDerivative(
            address _contractAddress,
            uint256 _weight
        ) external onlyOwner {

202:      function setMaxSlippage(
            uint _derivativeIndex,
            uint _slippage
        ) external onlyOwner {

214:     function setMinAmount(uint256 _minAmount) external onlyOwner {

223:     function setMaxAmount(uint256 _maxAmount) external onlyOwner {

232:     function setPauseStaking(bool _pause) external onlyOwner {

241:     function setPauseUnstaking(bool _pause) external onlyOwner {

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

107:     function withdraw(uint256 amount) external onlyOwner {

156:     function deposit() external payable onlyOwner returns (uint256) {

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

51:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

60:     function withdraw(uint256 _amount) external onlyOwner {

94:     function deposit() external payable onlyOwner returns (uint256) {

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

48:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

56:     function withdraw(uint256 _amount) external onlyOwner {

73:     function deposit() external payable onlyOwner returns (uint256) {

```

### [L-8] ACCOUNT EXISTENCE CHECK FOR LOW-LEVEL CALLS

#### Description:

Low-level calls `call`/`delegatecall`/`staticcall` return true even if the account called is non-existent (per EVM design). Account existence must be checked prior to calling if needed.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

76:         (bool sent, ) = WST_ETH.call{value: msg.value}("");

```

#### Recommended Mitigation Steps:

In addition to the zero-address checks, add a check to verify that <address>.code.length > 0

### [L-9] UNUSED `RECEIVE()` FUNCTION WILL LOCK ETHER IN CONTRACT

#### Description:

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

246:     receive() external payable {}

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

244:     receive() external payable {}

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

126:     receive() external payable {}

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

97:     receive() external payable {}

```

#### Recommended Mitigation Steps:

The function should call another function, otherwise it should revert

### [L-10] USE `_SAFEMINT` INSTEAD OF `_MINT`

#### Description:

According to openzepplin’s ERC721, the use of `_mint` is discouraged, use `safeMint` whenever possible.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-mint-address-uint256-

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

99:         _mint(msg.sender, mintAmount);

```

#### Recommended Mitigation Steps:

Use `_safeMint` whenever possible instead of `_mint`

### [L-11] USE `SAFETRANSFEROWNERSHIP` INSTEAD OF `TRANSFEROWNERSHIP` FUNCTION

#### Description:

transferOwnership function is used to change Ownership from Owned.sol.

Use a 2 structure transferOwnership which is safer.

safeTransferOwnership, use it is more secure due to 2-stage ownership transfer.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

53:         _transferOwnership(msg.sender);

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

43:         _transferOwnership(_owner);

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

37:         _transferOwnership(_owner);

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

34:         _transferOwnership(_owner);

```

#### Recommended Mitigation Steps:

Use Ownable2Step.sol
