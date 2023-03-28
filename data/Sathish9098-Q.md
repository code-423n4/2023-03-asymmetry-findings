# LOW FINDINGS 
##

## [L-1] UNUSED RECEIVE() functions

##

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity
97:  receive() external payable {}
```
[WstEth.sol#L97](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L97)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
```solidity 
126: receive() external payable {}
```
[SfrxEth.sol#L126](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L126)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
246: receive() external payable {}
````
[SafEth.sol#L246](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L246)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
244: receive() external payable {}
```
##

## [L-2] MISSING CHECKS FOR ADDRESS(0X0) WHEN TRANSFER OWNERSHIP 
##
Owner address is not checked before calling _transferOwnership function 

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity
function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
}
```

[WstEth.sol#L33-L36](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L33-L36)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
```solidity
function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
}
```
[SfrxEth.sol#L36-L39](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L36-L39)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
}
```
[Reth.sol#L42-L45](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L42-L45)

### Recommendations
##
Add necessary address(0) check before tranferownership to new address


##

## [L-3] Sanity/Threshold/Limit Checks

##
Devoid of sanity/threshold/limit checks, critical parameters can be configured to invalid values, causing a variety of issues and breaking expected interactions within/between contracts.

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity
 function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;

}
```
[WstEth.sol#L48-L50](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48-L50)

[WstEth.sol#L56-L67](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56-L67)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
```solidity
 function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
 }
```
[SfrxEth.sol#L51-L53](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
function setMaxAmount(uint256 _maxAmount) external onlyOwner {
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
}
```
[SafEth.sol#L223-L226](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L223-L226)
```solidity   
   function setMinAmount(uint256 _minAmount) external onlyOwner {
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }
```
[SafEth.sol#L214-L217](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L214-L217)
```solidity
     function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
     ) external onlyOwner {
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
     }
```
[SafEth.sol#L202-L208](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202-L208)

## Recommendations:

Add the sanity/threshold/limit checks before assigning critical variable values 

##

## [L-4] LOSS OF PRECISION DUE TO ROUNDING

##

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity
60: uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
```
[WstEth.sol#L60](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L60)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
```solidity
uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) * (10 ** 18 - maxSlippage)) / 10 ** 18;
```
[SfrxEth.sol#L74-L75](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75)
```solidity
    return ((10 ** 18 * frxAmount) / IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
```
[SfrxEth.sol#L115-L116](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L115-L116)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
81:  else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
```
[SafEth.sol#L81](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L81)
```solidity
88: uint256 ethAmount = (msg.value * weight) / totalWeight;
```
[SafEth.sol#L88](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L88)
```solidity
uint derivativeReceivedEthValue = (derivative.ethPerDerivative(depositAmount) * depositAmount) / 10 ** 18;
```
[SafEth.sol#L92-L94](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L92-L94)
```solidity
98: uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```
[SafEth.sol#L98](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L98)
```solidity

uint256 derivativeAmount = (derivatives[i].balance() * _safEthAmount) / safEthTotalSupply;
```
[SafEth.sol#L115-L116](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L115-L116)
```solidity
uint256 ethAmount = (ethAmountToRebalance * weights[i]) / totalWeight;
```
[SafEth.sol#L149-L150](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L149-L150)

##

## [L-5] A single point of failure

##

The onlyOwner role has a single point of failure and onlyOwner can use critical a few functions.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses

File: contracts/SafEth/SafEth.sol

```solidity

138:     function rebalanceToWeights() external onlyOwner {

168:     ) external onlyOwner {

185:     ) external onlyOwner {

205:     ) external onlyOwner {

214:     function setMinAmount(uint256 _minAmount) external onlyOwner {

223:     function setMaxAmount(uint256 _maxAmount) external onlyOwner {

232:     function setPauseStaking(bool _pause) external onlyOwner {

241:     function setPauseUnstaking(bool _pause) external onlyOwner {

```
File: contracts/SafEth/derivatives/Reth.sol

```solidity

58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

107:     function withdraw(uint256 amount) external onlyOwner {

156:     function deposit() external payable onlyOwner returns (uint256) {

```
File: contracts/SafEth/derivatives/SfrxEth.sol

```solidity

51:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

60:     function withdraw(uint256 _amount) external onlyOwner {

94:     function deposit() external payable onlyOwner returns (uint256) {

```
File: contracts/SafEth/derivatives/WstEth.sol

```solidity

48:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

56:     function withdraw(uint256 _amount) external onlyOwner {

73:     function deposit() external payable onlyOwner returns (uint256) {

```
##

## [L-6] Use safe variant of _mint() function

##

.mint won’t check if the recipient is able to receive the tokens. If an incorrect address is passed, it will result in a silent failure and loss of asset.

OpenZeppelin recommendation is to use the safe variant of _mint

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
99:  _mint(msg.sender, mintAmount);
```
[SafEth.sol#L99](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L99)

Recommendation
Replace _mint() with _safeMint()

##

## [L-7] No Storage Gap for SafEth contract

##

Impact
For upgradeable contracts, inheriting contracts may introduce new variables. In order to be able to add new variables to the upgradeable contract without causing storage collisions, a storage gap should be added to the upgradeable contract.

If no storage gap is added, when the upgradable contract introduces new variables, it may override the variables in the inheriting contract.

Storage gaps are a convention for reserving storage slots in a base contract, allowing future versions of that contract to use up those slots without affecting the storage layout of child contracts.
To create a storage gap, declare a fixed-size array in the base contract with an initial number of slots.
This can be an array of uint256 so that each element reserves a 32 byte slot. Use the naming convention __gap so that OpenZeppelin Upgrades will recognize the gap:

Storage Gaps
This makes the storage layouts incompatible, as explained in Writing Upgradeable Contracts. 
The size of the __gap array is calculated so that the amount of storage used by a contract 
always adds up to the same number (in this case 50 storage slots).

[SafEth.sol#L15-L20](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L15-L20)

## Recommended Mitigation Steps

##
Consider adding a storage gap at the end of the upgradeable abstract contract

uint256[50] private __gap;

##

## [L-8] Prevent division by 0

##

On several locations in the code precautions are not being taken for not dividing by 0, this will revert the code.
These functions can be called with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
88:  uint256 ethAmount = (msg.value * weight) / totalWeight;
```
[SafEth.sol#L88](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L88)
```solidity
98: uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```
[SafEth.sol#L98](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L98)
```solidity
115: uint256 derivativeAmount = (derivatives[i].balance() * _safEthAmount) / safEthTotalSupply;
```
[SafEth.sol#L115-L116](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L115-L116)
```solidity
149: uint256 ethAmount = (ethAmountToRebalance * weights[i]) /totalWeight;
```
[SafEth.sol#L149-L150)](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L149-L150)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
171: uint rethPerEth = (10 ** 36) / poolPrice();
```
[Reth.sol#L171](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L171)

##

## [L-9] Consider using OpenZeppelin’s SafeCast library to prevent unexpected behavior when casting uint

##

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
241: return (sqrtPriceX96 * (uint(sqrtPriceX96)) * (1e18)) >> (96 * 2);
```
[derivatives/Reth.sol#L241](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L241)

##

## [L-10] abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()

##

Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). "Unless there is a compelling reason, abi.encode should be preferred". If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead. If all arguments are strings and or bytes, bytes.concat() should be used instead

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
  RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
                keccak256(
                    abi.encodePacked("contract.address", "rocketTokenRETH")
                )
            );
```
[Reth.sol#L68-L72](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L68-L72)
```solidity
     address rocketDepositPoolAddress = RocketStorageInterface(
            ROCKET_STORAGE_ADDRESS
        ).getAddress(
                keccak256(
                    abi.encodePacked("contract.address", "rocketDepositPool")
                )
            );
```
[Reth.sol#L121-L127](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L121-L127)
```solidity
    address rocketProtocolSettingsAddress = RocketStorageInterface(
            ROCKET_STORAGE_ADDRESS
        ).getAddress(
                keccak256(
                    abi.encodePacked(
                        "contract.address",
                        "rocketDAOProtocolSettingsDeposit"
                    )
                )
            );
```
[Reth.sol#L132-L141](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L132-L141)
```solidity
     address rocketDepositPoolAddress = RocketStorageInterface(
            ROCKET_STORAGE_ADDRESS
        ).getAddress(
                keccak256(
                    abi.encodePacked("contract.address", "rocketDepositPool")
                )
            );
```
[Reth.sol#L158-L164](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L158-L164)
```solidity
    address rocketTokenRETHAddress = RocketStorageInterface(
                ROCKET_STORAGE_ADDRESS
            ).getAddress(
                    keccak256(
                        abi.encodePacked("contract.address", "rocketTokenRETH")
                    )
                );
```
[Reth.sol#L187-L193](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L187-L193)
```solidity
    address rocketTokenRETHAddress = RocketStorageInterface(
            ROCKET_STORAGE_ADDRESS
        ).getAddress(
                keccak256(
                    abi.encodePacked("contract.address", "rocketTokenRETH")
                )
            );
```
[Reth.sol#L229-L235](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L229-L235)

##

## [L-11] Missing Event for critical parameters init and change 

##

## Description
Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity
    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }
```
[WstEth.sol#L33-L36](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L33-L36)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
```solidity
    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }
```
[SfrxEth.sol#L36-L39](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L36-L39)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
    function initialize(
        string memory _tokenName,
        string memory _tokenSymbol
    ) external initializer {
        ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
        _transferOwnership(msg.sender);
        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
    }
```
[SafEth.sol#L48-L56](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L48-L56)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
   function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }
```
[Reth.sol#L42-L45](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L42-L45)

### Recommendation
Add Event-Emit

##

## [L-12] Gas griefing/theft is possible on unsafe external call

##

return data (bool success,) has to be stored due to EVM architecture, if in a usage like below, ‘out’ and ‘outsize’ values are given (0,0) . Thus, this storage disappears and may come from external contracts a possible Gas griefing/theft problem is avoided

<https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19>

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

```solidity

124: (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}("");

```
[SafEth.sol#L124-L126](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L124-L126)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

```solidity
76: (bool sent, ) = WST_ETH.call{value: msg.value}("");
63:  (bool sent, ) = address(msg.sender).call{value: address(this).balance}("");
```
[WstEth.sol#L76](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L76),[WstEth.sol#L63-L65](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L63-L65)

## recommanded Mitigations:

##
```solidity
assembly {                                    
                success := call(gas(), dest, amount, 0, 0)
         }   
```
##

## [L-13] Front running attacks by the onlyOwner

Attacker can front run the minAmount, maxAmount limitations. If minAmount increased still attacker can frontrun and stake their ETH with old values. Same way for maxAmount values

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

```solidity
function setMinAmount(uint256 _minAmount) external onlyOwner {
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }

function setMaxAmount(uint256 _maxAmount) external onlyOwner {
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
    }
```
[SafEth.sol#L214-L226](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L214-L226)

## Recommended Mitigation:
##
Use a timelock to avoid instant changes of the parameters

##

### [L-14] Function Calls in Loop Could Lead to Denial of Service

Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to the gas limitations or failed transactions. Here are some of the instances entailed.

<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L84>
<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113>
<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140>
<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L147>

##

# NON CRITICAL FINDINGS 

##

## [NC-1]  For modern and more readable code; update import usages

##

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity
import "../../interfaces/IDerivative.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../../interfaces/curve/IStEthEthPool.sol";
import "../../interfaces/lido/IWStETH.sol";
```
[WstEth.sol#L4-L8](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L4-L8)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
```solidity
import "../../interfaces/IDerivative.sol";
import "../../interfaces/frax/IsFrxEth.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../../interfaces/curve/IFrxEthEthPool.sol";
import "../../interfaces/frax/IFrxETHMinter.sol";
```
[SfrxEth.sol#L4-L9](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L4-L9)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../interfaces/IWETH.sol";
import "../interfaces/uniswap/ISwapRouter.sol";
import "../interfaces/lido/IWStETH.sol";
import "../interfaces/lido/IstETH.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "./SafEthStorage.sol";
import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
```
[SafEth.sol#L4-L11](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L4-L11)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
import "../../interfaces/IDerivative.sol";
import "../../interfaces/frax/IsFrxEth.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../../interfaces/rocketpool/RocketStorageInterface.sol";
import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
import "../../interfaces/IWETH.sol";
import "../../interfaces/uniswap/ISwapRouter.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "../../interfaces/uniswap/IUniswapV3Factory.sol";
import "../../interfaces/uniswap/IUniswapV3Pool.sol";
```
[Reth.sol#L4-L15](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L4-L15)

### Recommendation
##
import {contract1 , contract2} from "filename.sol";

##

## [NC-2] Imports can be grouped together

##

Consider importing interfaces first, then OpenZeppelin imports 

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity
import "../../interfaces/IDerivative.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../../interfaces/curve/IStEthEthPool.sol";
import "../../interfaces/lido/IWStETH.sol";
```
[WstEth.sol#L4-L8](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L4-L8)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
```solidity
import "../../interfaces/IDerivative.sol";
import "../../interfaces/frax/IsFrxEth.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../../interfaces/curve/IFrxEthEthPool.sol";
import "../../interfaces/frax/IFrxETHMinter.sol";
```
[SfrxEth.sol#L4-L9](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L4-L9)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../interfaces/IWETH.sol";
import "../interfaces/uniswap/ISwapRouter.sol";
import "../interfaces/lido/IWStETH.sol";
import "../interfaces/lido/IstETH.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "./SafEthStorage.sol";
import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
```
[SafEth.sol#L4-L11](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L4-L11)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
import "../../interfaces/IDerivative.sol";
import "../../interfaces/frax/IsFrxEth.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../../interfaces/rocketpool/RocketStorageInterface.sol";
import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
import "../../interfaces/IWETH.sol";
import "../../interfaces/uniswap/ISwapRouter.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "../../interfaces/uniswap/IUniswapV3Factory.sol";
import "../../interfaces/uniswap/IUniswapV3Pool.sol";
```
[Reth.sol#L4-L15](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L4-L15)


 
##

## [NC-3] AVOID HARDCODED VALUES 

##

It is not good practice to hardcode values, but if you are dealing with addresses much less, these can change between implementations, networks or projects, so it is convenient to remove these values from the source code

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

```solidity
address public constant WST_ETH = 0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0;
address public constant LIDO_CRV_POOL = 0xDC24316b9AE028F1497c275EB9192a3Ea0f67022;
address public constant STETH_TOKEN = 0xae7ab96520DE3A18E5e111B5EaAb095312D7fE84
```
[WstEth.sol#L13-L18](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L13-L18)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
 ```solidity
 address public constant SFRX_ETH_ADDRESS = 0xac3E018457B222d93114458476f3E3416Abbe38F;
 address public constant FRX_ETH_ADDRESS = 0x5E8422345238F34275888049021821E8E08CAa1f;
 address public constant FRX_ETH_CRV_POOL_ADDRESS = 0xa1F8A6807c402E4A15ef4EBa36528A3FED24E577;
 address public constant FRX_ETH_MINTER_ADDRESS = 0xbAFA44EFE7901E04E39Dad13167D089C559c1138;
```
[SfrxEth.sol#L14-L21](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L14-L21)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
address public constant ROCKET_STORAGE_ADDRESS = 0x1d8f8f00cfa6758d7bE78336684788Fb0ee0Fa46;
address public constant W_ETH_ADDRESS = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;
address public constant UNISWAP_ROUTER = 0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45;
address public constant UNI_V3_FACTORY = 0x1F98431c8aD98523631AE4a59f267346ea31F984;
```
[Reth.sol#L20-L27](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L20-L27)

##

## [NC-4] Shorter the inheritance 

##

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
   contract SafEth is
    Initializable,
    ERC20Upgradeable,
    OwnableUpgradeable,
    SafEthStorage
```
[SafEth/SafEth.sol#L15-L19](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L15-L19)

##

## [NC-5] EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING

##

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity
97:  receive() external payable {}
```
[WstEth.sol#L97](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L97)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
 ```solidity
126: receive() external payable {}
```
[SfrxEth.sol#L126](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L126)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
246: receive() external payable {}
```
[SafEth.sol#L246](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L246)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
244: receive() external payable {}
```
##

## [NC-6] ADD PARAMETER TO EVENT-EMIT

##

Some event-emit description hasn’t parameter. Add to parameter for front-end website or client app , they can has that something has happened on the blockchain

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
34: event Rebalanced();
```
[SafEth.sol#L34](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L34)

##

## [NC-7] Pragma float

##

All the contracts in scope are floating the pragma version.

### Recommendation

## 

Locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

Note that pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or a package.

##

## [NC-8] Interchangeable usage of uint and uint256

<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L91-L92>
<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L171-L173>

##

```solidity
  function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
    ) external onlyOwner {
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
    }
```
[SafEth.sol#L202-L208](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202-L208)

##

## [NC-9] TYPOS

##

FILE: 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```
/// @audit derivates => derivatives 
- 160:    @dev - If you want exact weights either do the math off chain or reset all existing derivates to the weights you want
+ 160:    @dev - If you want exact weights either do the math off chain or reset all existing derivatives to the weights you want
```
##

## [NC-10] Include (@param and @return) parameters in NatSpec comments

##
Context
All Contracts

Description
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

[natspec-format.html](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)

## Recommendation
##
Include return and function arguments(@param,@return) parameters in NatSpec comments

Recommendation Code Style: (from Uniswap3)
```
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

##

## [NC-11] NatSpec comments should be increased in contracts

##

### Context
All Contracts

### Description:
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.

In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. 

<https://docs.soliditylang.org/en/v0.8.15/natspec-format.html>

### Recommendation
##
NatSpec comments should be increased in contracts

##

## [NC-12] Function writing that does not comply with the Solidity Style Guide

##

### Context
All Contracts

Description
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

<https://docs.soliditylang.org/en/v0.8.17/style-guide.html>

Functions should be grouped according to their visibility and ordered:

- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- within a grouping, place the view and pure functions last

##

## [N-13] Add a timelock to critical functions

##
It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). Consider adding a timelock to

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity
 function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
}
```
[WstEth.sol#L48-L50](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48-L50)

[WstEth.sol#L56-L67](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56-L67)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
```solidity
 function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
 }
```
[SfrxEth.sol#L51-L53](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
function setMaxAmount(uint256 _maxAmount) external onlyOwner {
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
}
```
[SafEth.sol#L223-L226](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L223-L226)
```solidity   
   function setMinAmount(uint256 _minAmount) external onlyOwner {
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }
```
[SafEth.sol#L214-L217](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L214-L217)
```solidity
     function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
     ) external onlyOwner {
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
     }
```
[SafEth.sol#L202-L208](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202-L208)

##

## [NC-14] For functions, follow Solidity standard naming conventions (private function style rule)

##

## Description
The above codes don’t follow Solidity’s standard naming convention,

For private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol

```solidity

66: function rethAddress() private view returns (address) {

function swapExactInputSingleHop(
        address _tokenIn,
        address _tokenOut,
        uint24 _poolFee,
        uint256 _amountIn,
        uint256 _minOut
) private returns (uint256 amountOut) {

120:  function poolCanDeposit(uint256 _amount) private view returns (bool) {

228:  function poolPrice() private view returns (uint256) {

```   

##

## [NC-15] Missing NatSpec

Consider adding specification to the following code blocks

##

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

```solidity 
event ChangeMinAmount(uint256 indexed minAmount);
event ChangeMaxAmount(uint256 indexed maxAmount);
event StakingPaused(bool indexed paused);
event UnstakingPaused(bool indexed paused);
event SetMaxSlippage(uint256 indexed index, uint256 slippage);
event Staked(address indexed recipient, uint ethIn, uint safEthOut);
event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);
event WeightChange(uint indexed index, uint weight);
event DerivativeAdded(
        address indexed contractAddress,
        uint weight,
        uint index
    );
event Rebalanced();
```
[SafEth.sol#L21-L34](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L21-L34)

##

### [NC-16] Use scientific notation instead of exponential notation

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

```solidity

35: maxSlippage = (1 * 10 ** 16); // 1%
60: uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
87: return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
```
[WstEth.sol#L35](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L35),[WstEth.sol#L60](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L60),[WstEth.sol#L87](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L87)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol

```solidity

38: maxSlippage = (1 * 10 ** 16); // 1%
74: uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
            (10 ** 18 - maxSlippage)) / 10 ** 18;
112: uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
            10 ** 18
        );
116: return ((10 ** 18 * frxAmount) /
            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
```
[SfrxEth.sol#L38](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L38),[SfrxEth.sol#L74-L75](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75),[SfrxEth.sol#L112-L116](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L112-L116)

FILE: 2023-03-asymmetry/contracts/SafEth/SafEth.sol

```solidity
75:  10 ** 18;
80:  preDepositPrice = 10 ** 18; // initializes with a price of 1
81:  else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
94:  ) * depositAmount) / 10 ** 18;
98:  uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```

[SafEth.sol#L75](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L75),[SafEth.sol#L80-L81](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L80-L81),[SafEth.sol#L94](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L94),[SafEth.sol#L98](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L98)

##

## [NC-17] String constants used in multiple places should be defined as constants

contract.address and rocketTokenRETH should be defined rathar than using multiple times 

<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L136-L139>
<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L125>
<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L70>
<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L162>
<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L191>
<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L233>

##

## [NC-18] Events that mark critical parameter changes should contain both the old and the new value

This should especially be done if the new value is not required to be different from the old value

<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L232-L244>

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
function setMaxAmount(uint256 _maxAmount) external onlyOwner {
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
}
```
[SafEth.sol#L223-L226](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L223-L226)
```solidity   
   function setMinAmount(uint256 _minAmount) external onlyOwner {
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }
```
[SafEth.sol#L214-L217](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L214-L217)
```solidity
     function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
     ) external onlyOwner {
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
     }
```
[SafEth.sol#L202-L208](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202-L208)

##

## [N-19] NO SAME VALUE INPUT CONTROL 

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
function setMaxAmount(uint256 _maxAmount) external onlyOwner {
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
}
```
[SafEth.sol#L223-L226](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L223-L226)
```solidity   
   function setMinAmount(uint256 _minAmount) external onlyOwner {
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }
```
[SafEth.sol#L214-L217](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L214-L217)
```solidity
     function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
     ) external onlyOwner {
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
     }
```
[SafEth.sol#L202-L208](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202-L208)

<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L232-L244>








NC-1	Return values of approve() not checked	3
NC-2	Event is missing indexed fields	5
NC-3	Constants should be defined rather than using magic numbers	1
NC-4	Functions not used internally could be marked external	8
L-1	Empty Function Body - Consider commenting why	4
L-2	Initializers could be front-run	9
L-3	Unsafe ERC20 operation(s)	3