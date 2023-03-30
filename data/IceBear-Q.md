## Non Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [N-1](#N-1) | Large multiples of ten should use scientific notation rather than decimal literals, for readability | 21 |
| [N-2](#N-2) | Unused imports | 6 |
| [N-3](#N-3) | Unused parameters | 2 |
| [N-4](#N-4) | For modern and more readable code; update import usages | 31 |

## Low Issues

| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) |Use ownable2stepupgradeable instead of ownableupgradeable contract | 4 |
| [L-2](#L-2) | Non-library/interface files should use fixed compiler versions, not floating ones | 4 |
| [L-3](#L-3) | Unused/Empty RECEIVE()/FALLBACK() function | 4 |
| [L-4](#L-4) | Use of hard-coded addresses may cause errors | 11 |
| [L-5](#L-5) | initialize() function can be called by anybody | 7 |
|[L-6](#L-6) | No storage gap for upgradeable contract might lead to storage slot collision | 5 |
|[L-7](#L-7) | Lack of events emission after sensitive actions | 9 |

### [N-1] Large multiples of ten should use scientific notation rather than decimal literals, for readability
#### Recommendation
Scientific notation should be used for better code readability.
Use scientific notation (e.g. 1e18, 1e5) rather than exponentiation (e.g. 10**18, 100000).

*Find (21) instance(s) in contracts*:
```solidity
File: SafEth/SafEth.sol

54:         minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum

55:         maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum

75:                 10 ** 18;

80:             preDepositPrice = 10 ** 18; // initializes with a price of 1

81:         else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

94:             ) * depositAmount) / 10 ** 18;

98:         uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;

```
[SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol)

```solidity
File: SafEth/derivatives/Reth.sol

44:         maxSlippage = (1 * 10 ** 16); // 1%

171:             uint rethPerEth = (10 ** 36) / poolPrice();

173:             uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *

174:                 ((10 ** 18 - maxSlippage))) / 10 ** 18);

214:                 RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);

215:         else return (poolPrice() * 10 ** 18) / (10 ** 18);

```
[SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol)

```solidity
File: SafEth/derivatives/SfrxEth.sol

38:         maxSlippage = (1 * 10 ** 16); // 1%

74:         uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *

75:             (10 ** 18 - maxSlippage)) / 10 ** 18;

113:             10 ** 18

115:         return ((10 ** 18 * frxAmount) /

```
[SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol)

```solidity
File: SafEth/derivatives/WstEth.sol

35:         maxSlippage = (1 * 10 ** 16); // 1%

60:         uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;

87:         return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);

```
[SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol)



### [N-2] Unused imports

*Find (6) instance(s) in contracts*:
```solidity
File: SafEth/SafEth.sol

4: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

5: import "../interfaces/IWETH.sol";

6: import "../interfaces/uniswap/ISwapRouter.sol";

7: import "../interfaces/lido/IWStETH.sol";

8: import "../interfaces/lido/IstETH.sol";

```
[SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol)

```solidity
File: SafEth/derivatives/Reth.sol

5: import "../../interfaces/frax/IsFrxEth.sol";

```
[SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol)

### [N-3] Unused parameters

*Find (2) instance(s) in contracts*:
```solidity
File: SafEth/derivatives/SfrxEth.sol

111:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {

```
[SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol)

```solidity
File: SafEth/derivatives/WstEth.sol

86:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {

```
[SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol)


### [N-4] For modern and more readable code; update import usages

Our Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it. This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

*Find (31) instance(s) in contracts*:
```solidity
File: SafEth/derivatives/WstEth.sol

4 import "../../interfaces/IDerivative.sol";
5 import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
6 import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
7 import "../../interfaces/curve/IStEthEthPool.sol";
8:import "../../interfaces/lido/IWStETH.sol";

```
[SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol)

```solidity
File: SafEth/derivatives/SfrxEth.sol
  
4 import "../../interfaces/IDerivative.sol";
5 import "../../interfaces/frax/IsFrxEth.sol";
6 import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
7 import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
8 import "../../interfaces/curve/IFrxEthEthPool.sol";
9:import "../../interfaces/frax/IFrxETHMinter.sol";

```
[SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol)

```solidity
File: SafEth/SafEth.sol
  
4 import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
5 import "../interfaces/IWETH.sol";
6 import "../interfaces/uniswap/ISwapRouter.sol";
7 import "../interfaces/lido/IWStETH.sol";
8 import "../interfaces/lido/IstETH.sol";
9 import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
10 import "./SafEthStorage.sol";
11:import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

```
[SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)

```solidity
File: SafEth/derivatives/Reth.sol
  
4 import "../../interfaces/IDerivative.sol";
5 import "../../interfaces/frax/IsFrxEth.sol";
6 import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
7 import "../../interfaces/rocketpool/RocketStorageInterface.sol";
8 import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
9 import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
10 import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
11 import "../../interfaces/IWETH.sol";
12 import "../../interfaces/uniswap/ISwapRouter.sol";
13 import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
14 import "../../interfaces/uniswap/IUniswapV3Factory.sol";
15:import "../../interfaces/uniswap/IUniswapV3Pool.sol";
```
[SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol)


#### Recommendation: 
import {contract1 , contract2} from "filename.sol";

### [L-1] Use ownable2stepupgradeable instead of ownableupgradeable contract
transferOwnership function is used to change Ownership from OwnableUpgradeable.sol.

There is another Openzeppelin Ownable contract (Ownable2StepUpgradeable.sol) has transferOwnership function , use it is more secure due to 2-stage ownership transfer.

*Find (4) instance(s) in contracts*:
```solidity
File: SafEth/SafEth.sol

9: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

```
[SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol)

```solidity
File: SafEth/derivatives/Reth.sol

13: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

```
[SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol)

```solidity
File: SafEth/derivatives/SfrxEth.sol

6: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

```
[SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol)

```solidity
File: SafEth/derivatives/WstEth.sol

5: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

```
[SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol)



### [L-2] Non-library/interface files should use fixed compiler versions, not floating ones 

*Find (4) instance(s) in contracts*:
```solidity
File: SafEth/SafEth.sol

2: pragma solidity ^0.8.13;

```
[SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol)

```solidity
File: SafEth/derivatives/Reth.sol

2: pragma solidity ^0.8.13;

```
[SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol)

```solidity
File: SafEth/derivatives/SfrxEth.sol

2: pragma solidity ^0.8.13;

```
[SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol)

```solidity
File: SafEth/derivatives/WstEth.sol

2: pragma solidity ^0.8.13;

```
[SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol)

### [L-3] Unused/Empty RECEIVE()/FALLBACK() function
If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))). Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds.

*Find (4) instance(s) in contracts*:
```solidity
File: SafEth/SafEth.sol

246:     receive() external payable {}

```
[SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol)

```solidity
File: SafEth/derivatives/Reth.sol

244:     receive() external payable {}

```
[SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol)

```solidity
File: SafEth/derivatives/SfrxEth.sol

126:     receive() external payable {}

```
[SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol)

```solidity
File: SafEth/derivatives/WstEth.sol

97:     receive() external payable {}

```
[SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol)

### [L-4] Use of hard-coded addresses may cause errors

These addresses are currently hard-coded, which may cause errors and result in the codebase’s deployment with an incorrect asset. Using hard-coded values instead of deployer-provided values makes these contracts incredibly difficult to test.
similar finding:
https://github.com/trailofbits/publications/blob/master/reviews/AdvancedBlockchain.pdf


*Find (11) instance(s) in contracts*:
```solidity
File: SafEth/derivatives/Reth.sol

20:     address public constant ROCKET_STORAGE_ADDRESS =

22:     address public constant W_ETH_ADDRESS =

24:     address public constant UNISWAP_ROUTER =

26:     address public constant UNI_V3_FACTORY =

```
[SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol)

```solidity
File: SafEth/derivatives/SfrxEth.sol

14:     address public constant SFRX_ETH_ADDRESS =

16:     address public constant FRX_ETH_ADDRESS =

18:     address public constant FRX_ETH_CRV_POOL_ADDRESS =

20:     address public constant FRX_ETH_MINTER_ADDRESS =

```
[SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol)

```solidity
File: SafEth/derivatives/WstEth.sol

13:     address public constant WST_ETH =

15:     address public constant LIDO_CRV_POOL =

17:     address public constant STETH_TOKEN =

```
[SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol)

#### Recommendation
Short term, set addresses when contracts are created rather than using hard-coded values. This practice will facilitate testing. Long term, to ensure that contracts can be tested and reused across networks, avoid using hard-coded parameters.

### [L-5] initialize() function can be called by anybody

initialize() function can be called anybody when the contract is not initialized.
Also, there is no 0 address check in the address arguments of the initialize() function, which must be defined.


*Find (7) instance(s) in contracts*:
```solidity
File: SafEth/SafEth.sol

48:     function initialize(

```
[SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol)

```solidity
File: SafEth/derivatives/Reth.sol

42:     function initialize(address _owner) external initializer {

42:     function initialize(address _owner) external initializer {

```
[SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol)

```solidity
File: SafEth/derivatives/SfrxEth.sol

36:     function initialize(address _owner) external initializer {

36:     function initialize(address _owner) external initializer {

```
[SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol)

```solidity
File: SafEth/derivatives/WstEth.sol

33:     function initialize(address _owner) external initializer {

33:     function initialize(address _owner) external initializer {

```
[SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol)


#### Recommendation 
Add a control that makes initialize() only call the Deployer Contract;
```
if (msg.sender != DEPLOYER_ADDRESS) {
						revert NotDeployer();
				}
```
### [L-6]No storage gap for upgradeable contract might lead to storage slot collision

For upgradeable contracts, there must be storage gap to “allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments” (quote OpenZeppelin). Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts, potentially causing loss of user fund or cause the contract to malfunction completely.

Refer to the bottom part of this article: https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable


*Find (5) instance(s) in contracts*:
```solidity
File: SafEth/SafEth.sol

9:     import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

11:    import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

```
[SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol)

```solidity
File: SafEth/derivatives/Reth.sol

13:     import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
```
[SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol)

```solidity
File: SafEth/derivatives/SfrxEth.sol

6:     import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
```
[SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol)

```solidity
File: SafEth/derivatives/WstEth.sol

5:     import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
```
[SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol)

#### Recommendation
Recommend adding appropriate storage gap at the end of upgradeable contracts such as the below. Please reference OpenZeppelin upgradeable contract templates.
```
uint256[50] private __gap;
```

### [L-7]Lack of events emission after sensitive actions

There are several cases where sensitive actions are performed but there are no events being emitted.

similar findings:
-(M01) https://blog.openzeppelin.com/uma-audit-phase-4/
-(M10) https://blog.openzeppelin.com/holdefi-audit/#medium


*Find (9) instance(s) in contracts*:

```solidity
File: SafEth/derivatives/Reth.sol

58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

107:    function withdraw(uint256 amount) external onlyOwner {
    
156:    function deposit() external payable onlyOwner returns (uint256) {
```
[SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol)

```solidity
File: SafEth/derivatives/SfrxEth.sol

51:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
    
60:    function withdraw(uint256 _amount) external onlyOwner {
    
94:    function deposit() external payable onlyOwner returns (uint256) {
```
[SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol)

```solidity
File: SafEth/derivatives/WstEth.sol

48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
    
56:    function withdraw(uint256 _amount) external onlyOwner {
    
73:    function deposit() external payable onlyOwner returns (uint256) {
```
[SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol)

#### Recommendation
Consider emitting events after sensitive changes take place, to facilitate tracking and notify off-chain clients following the contract’s activity.