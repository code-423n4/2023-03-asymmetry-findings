# Report
## Low Risk ##
### [L-1]: Check if user has enough safETH
**Context:**

```function unstake(uint256 _safEthAmount) external {``` [L108](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108)

**Recommendation:**
add this to the beginning of the function:
```
require(balanceOf(msg.sender) >= _safEthAmount, "not enough tokens");
```

### [L-2]: Use Ownable2StepUpgradeable instead of  OwnableUpgradeable contract
**Context:**

1. ```import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";``` [L5](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L5) 
1. ```import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";``` [L6](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L6) 
1. ```import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";``` [L9](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L9) 
1. ```import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";``` [L13](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L13) 

**Description:**

There is another Openzeppelin Ownable contract [Ownable2StepUpgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol). It also has transferOwnership function, but it is more secure due to 2-stage ownership transfer.

### [L-3]: Misleading comments
**Context:**

1. ```@param _owner - owner of the contract which handles stake/unstake``` [L31](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L31)
1. ```@param _owner - owner of the contract which handles stake/unstake``` [L40](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L40)
1. ```@param _owner - owner of the contract which handles stake/unstake``` [L34](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L34)

**Recommendation:**
Change to ```@param _owner - address of SafEth contract```.

### [L-4]: Critical changes should use two-step procedure
**Context:**

1. ```function setMaxSlippage(``` [L201](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L201) 
1. ```function setMinAmount(uint256 _minAmount) external onlyOwner {``` [L213](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L213) 
1. ```function setMaxAmount(uint256 _maxAmount) external onlyOwner {``` [L222](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L222) 
1. ```function setPauseStaking(bool _pause) external onlyOwner {``` [L231](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L231) 
1. ```function setPauseUnstaking(bool _pause) external onlyOwner {``` [L240](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L240) 

**Recommendation:**

The best practice is to use two-step procedure for critical changes to make them less error-prone. 

### [L-5]: Owner can renounce ownership
**Context:**

1. ```import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";``` [L5](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L5) 
1. ```import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";``` [L6](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L6) 
1. ```import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";``` [L9](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L9) 
1. ```import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";``` [L13](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L13)

**Description:**

"@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol" used in this project contracts implements renounceOwnership() function. Renouncing ownership will leave the contract without an owner, protected functions may become permanently inaccessible.

**Recommendation:**

You need to reimplement the function.

### [L-6]: Variable is unused
**Context:**

1. ```function ethPerDerivative(uint256 _amount) public view returns (uint256) {``` [L86](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86) (_amount)
1. ```function ethPerDerivative(uint256 _amount) public view returns (uint256) {``` [L111](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L111) (_amount)

**Description:**
this variable is not actually used in the functions. Forcing users to enter a value can mislead them.

**Recommendation:**
Write a comment that the variable is not used.

### [L-7]: Incorrect function description
**Context:**

```@notice - Adds new derivative to the index fund``` [L158](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L158)

**Description:**
Misleading comment, function does not add new derivative, it updates the weight.

### [L-8]: Add return parameter in stake() function

**Context:**
1. ```function stake() external payable {``` [L63](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63)

**Description:**
msg.value is not equal to the number of received tokens. Users will be more convenient to see the number of the received tokens.

**Recommendation:**
Add ```return mintAmount;``` at the end of the function.

### [L-9]: Omissions in Events
**Context:**

1. ```event ChangeMinAmount(uint256 indexed minAmount);``` [L21](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L21) (the events should include the new value and old value where possible)
1. ```event ChangeMaxAmount(uint256 indexed maxAmount);``` [L22](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L22) (the events should include the new value and old value where possible)
1. ```event WeightChange(uint indexed index, uint weight);``` [L28](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L28) (totalWeight changes too, new totalWeight must be included in event)
1. ```event DerivativeAdded(;``` [L29](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L29) (totalWeight changes too, new totalWeight must be included in event)

**Description:**
Events are generally emitted when sensitive changes are made to the contracts. Some events are missing important parameters.

### [L-10]: Check if already paused/not paused

**Context:**

1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L224
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L233

**Description:**
The owner may accidentally set the same value of pauseStaking/pauseUnstaking that has already been set, and not notice it for a long time. This can lead to disastrous consequences.
**Recommendation:**
To avoid this, you can add checks:  
1. require(pauseStaking != _pause)
1. require(pauseUnstaking != _pause)

## Non-Critical Issues ##
### [N-1]: NatSpec is incomplete
**Context:** 
1. ```function setMaxSlippage(uint256 _slippage) external onlyOwner {```[L48](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48) (_slippage)
1. ```function withdraw(uint256 _amount) external onlyOwner {```[L56](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56) (_amount)
1. ```function ethPerDerivative(uint256 _amount) public view returns (uint256) {```[L86](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86) (_amount)
1. ```function setMaxSlippage(uint256 _slippage) external onlyOwner {```[L51](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51) (_slippage)
1. ```function ethPerDerivative(uint256 _amount) public view returns (uint256) {```[L111](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L111) (_amount)
1. ```function withdraw(uint256 amount) external onlyOwner {```[107](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107) (_amount)


### [N-2]: Unnecessary code splitting
**Context:** 
```
                    derivatives[i].balance()) /
                10 ** 18;
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L74-L75
Change to:
```
                    derivatives[i].balance()) / 10 ** 18;
```

```
        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
            ""
        );
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L124-L126
Change to:
```
        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}("");
```

Also here:
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L92-L94
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L115-L116
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L141-L142
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L149-L150
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L66-L68
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L84-L86
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L95-L97
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L98-L100
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L102-L104
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L112-L114
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L115-L116
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L110-L112
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L128-L130
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L166-L168
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173-L174
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L194-L196
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L213-L214
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L237-L239


### [N-3]: Wrong order of functions
**Context:**

1. ```function setMaxSlippage(uint256 _slippage) external onlyOwner {``` [L48](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48) (external function can not go after public function)
1. ```receive() external payable {}``` [L97](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L97) (receive function can not go after public function)
1. ```function setMaxSlippage(uint256 _slippage) external onlyOwner {``` [L51](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51) (external function can not go after public function)
1. ```receive() external payable {}``` [L126](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L126) (receive function can not go after public function)
1. ```receive() external payable {}``` [L245](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L245) (receive function can not go after external function)
1. ```function setMaxSlippage(uint256 _slippage) external onlyOwner {``` [L58](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58) (external function can not go after public function)
1. ```receive() external payable {}``` [L244](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L244) (receive function can not go after public function)

**Description:**

According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) functions should be grouped according to their visibility and ordered:

+ constructor

+ receive function (if exists)

+ fallback function (if exists)

+ external

+ public

+ internal

+ private

Within a grouping, place the view and pure functions last.

**Recommendation:**

Put the functions in the correct order according to the documentation.

### [N-4]: Change imports
**Context:**

1. ```import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";``` [L5](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L5) 
1. ```import "@openzeppelin/contracts/token/ERC20/IERC20.sol";``` [L6](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L6) 
1. ```import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";``` [L6](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L6) 
1. ```import "@openzeppelin/contracts/token/ERC20/IERC20.sol";``` [L7](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L7) 
1. ```import "@openzeppelin/contracts/token/ERC20/IERC20.sol";``` [L4](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L4)
1. ```import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";``` [L9](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L9) 
1. ```import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";``` [L11](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L11) 
1. ```import "@openzeppelin/contracts/token/ERC20/IERC20.sol";``` [L6](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L6) 
1. ```import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";``` [L13](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L13) 

**Recommendation:**
Example:
Instead of:
```import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";```

You can do your imports like this:
```import {OwnableUpgradeable} from "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol"```;

### [N-5]: Missing leading underscores
**Context:**

1. ```function rethAddress() private view returns (address) {``` [L66](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L66) 
1. ```function swapExactInputSingleHop(``` [L83](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L83) 
1. ```function poolCanDeposit(uint256 _amount) private view returns (bool) {``` [L120](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L120) 
1. ```function poolPrice() private view returns (uint256) {``` [L228](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L228) 

**Description:**
Internal and private functions, state variables, constants, and immutables should starting with an underscore.


### [N-6]: Line is too long
**Context:**

```RocketDAOProtocolSettingsDepositInterface rocketDAOProtocolSettingsDeposit = RocketDAOProtocolSettingsDepositInterface(``` [L142](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L142) 

**Description:**

Maximum suggested line length is [120 characters](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length).


### [N-7]: Typo in event name
**Context:**

```event WeightChange(uint indexed index, uint weight);``` [L28](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L28) (Change **WeightChange** to **WeightChanged**)


### [N-8]: Emit events in initialize() functions
##### Instances
1. ```function initialize(address _owner) external initializer {``` [L33](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L33)
1. ```function initialize(address _owner) external initializer {``` [L36](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L36)
1. ```function initialize(address _owner) external initializer {``` [L42](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L42)
1. ```function initialize(``` [L48](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L48)

