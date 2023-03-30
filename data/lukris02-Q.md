# QA Report for Asymmetry contest
## Overview
During the audit, 12 low and 12 non-critical issues were found.

№ | Title | Risk Rating  | Instance Count
--- | --- | --- | ---
L-1 | Use the two-step-transfer of ownership | Low | 4
L-2 | Critical changes should use two-step procedure | Low | 6
L-3 | Owner can renounce ownership | Low | 4
L-4 | Add a timelock to critical functions | Low | 6
L-5 | Misleading comment about owner | Low | 3
L-6 | Add return parameter in ```stake()``` | Low | 1
L-7 | Check ```mintAmount``` | Low | 1
L-8 | Check user token balance | Low | 1
L-9 | Add more information in the events | Low | 5
L-10 | Set ```minMinAmount``` | Low | 1
L-11 | Make separate events | Low | 2
L-12 | Unused variable in the ```ethPerDerivative()``` is misleading | Low | 2
NC-1 | Missing leading underscores | Non-Critical | 4
NC-2 | Order of Functions | Non-Critical | 11
NC-3 | Typo in event name | Non-Critical | 1
NC-4 | Constants may be used | Non-Critical | 17
NC-5 | Make event names more consistent | Non-Critical | 2
NC-6 | Emit events in initialize() functions | Non-Critical | 4
NC-7 | Put ```deposit()``` before ```withdraw()``` | Non-Critical | 3
NC-8 | Concatenate multiple lines of code | Non-Critical | 11
NC-9 | Incorrect comment | Non-Critical | 1
NC-10 | Make commentss more consistent | Non-Critical | 2
NC-11 | Natspec is incomplete | Non-Critical | 6
NC-12 | Inconsistency when using uint and uint256 | Non-Critical | 2

## Low Risk Findings(12)
### L-1. Use the two-step-transfer of ownership
##### Description
If the owner accidentally transfers ownership to an incorrect address, protected functions may become permanently inaccessible.
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L5
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L6
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L13
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L9

##### Recommendation
Consider using a two-step-transfer of ownership: the current owner would nominate a new owner, and to become the new owner, the nominated account would have to approve the change, so that the address is proven to be valid.
#
### L-2. Critical changes should use two-step procedure
##### Description
Lack of two-step procedure for critical operations leaves them error-prone.
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223

##### Recommendation
Consider adding two step procedure on the critical functions.
#
### L-3. Owner can renounce ownership
##### Description
Openzeppelin's OwnableUpgradeable.sol implements ```renounceOwnership()``` function which leaves the contract without an owner and removes any functionality that is only available to them.
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L5
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L6
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L13
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L9

##### Recommendation
Consider reimplementing the function to disable it.
#
### L-4. Add a timelock to critical functions
##### Description
Giving users time to react and adjust to critical changes in protocol provides more guarantees and increases the transparency of the protocol.

##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223

##### Recommendation
Consider adding a timelock.
#
### L-5. Misleading comment about owner
##### Description
Comment for the initialize() function says that parameter ```owner``` is a "owner of the contract which handles stake/unstake", although in fact the SafEth contract itself must be specified as the owner.
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L31
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L34
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L40

##### Recommendation
Change the comments to "address of SafEth contract"
#
### L-6. Add return parameter in ```stake()```
##### Description
Since msg.value is not the same as the received number of tokens, for more convenience and clarity, it is better to add the return parameter in the stake() function.

##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63-L101

##### Recommendation
Add at the end of the function:
```return mintAmount;```
#
### L-7. Check ```mintAmount```
##### Description
To prevent cases when the user pays eth but gets nothing, it should be checked that they receive at least 1 SafEth. Such cases are possible if the protocol lowers the minimum amount of Ether for staking in the future.

##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L99

##### Recommendation
```require(mintAmount > 0, "no tokens");```. Or even better, add a parameter so that users can specify the minimum number of tokens they want to receive.
#
### L-8. Check user token balance
##### Description
At the beginning of the unstake() function, it is necessary to check that the user has enough tokens on the balance.
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L109

##### Recommendation
```require(balanceOf(msg.sender) >= _safEthAmount, "not enough tokens");```
#
### L-9. Add more information in the events
##### Description
Some events are missing important information.
##### Instances
- [```emit WeightChange(_derivativeIndex, _weight);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L174) => add ```totalWeight```
- [```emit DerivativeAdded(_contractAddress, _weight, derivativeCount);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L194) => add ```totalWeight```
- [```emit SetMaxSlippage(_derivativeIndex, _slippage);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L207) => add ```previous slippage```
- [```emit ChangeMinAmount(minAmount);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L216) => add ```previous minAmount```
- [```emit ChangeMaxAmount(maxAmount);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L225) => add ```previous maxAmount```

#
### L-10. Set ```minMinAmount```
##### Description
Set the minimum possible ```minAmount``` to ensure that the owner cannot set a value that is too small, which could negatively affect the received tokens due to roundings. This will enhance the reliability and security of the protocol.
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L215

##### Recommendation
Add a constant ```minMinAmount```, and in the setMinAmount(), ```require(_minAmount >= minMinAmount, "too small value");```
#
### L-11. Make separate events
##### Description
It is better to make separate events "StakingPaused" and "StakingUnpaused" ("UnstakingPaused" and "UnstakingUnpaused") because the meaning of events will be more obvious, and there will be no need to remember what 0 and 1 means. 

##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L234
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L243

#
### L-12. Unused variable in the ```ethPerDerivative()``` is misleading
##### Description
2 out of 3 derivatives do not use the ```amount``` parameter in the ethPerDerivative function, while the third derivative does. This can be confusing because, without documentation, it is not immediately obvious that the function calculates the price in Ether for only one unit of the derivative, and not for a given ```amount```.

##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L111

##### Recommendation
For clarity, add comments that clearly state what the ```amount``` passed in is used for, and that the price is calculated for only one unit. 
#
## Non-Critical Risk Findings(12)
### NC-1. Missing leading underscores
##### Description
Private functions should have a leading underscore.
##### Instances
- [```function rethAddress() private view returns (address) {```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L66) 
- [```function swapExactInputSingleHop(```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L83) 
- [```function poolCanDeposit(uint256 _amount) private view returns (bool) {```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L120) 
- [```function poolPrice() private view returns (uint256) {```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L228) 

##### Recommendation
Add leading underscores where needed.
#
### NC-2. Order of Functions
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-functions), ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.  
Functions should be grouped according to their visibility and ordered:
1) constructor
2) receive function (if exists)
3) fallback function (if exists)
4) external
5) public
6) internal
7) private
##### Instances
receive() function should be placed right after constructor:
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L97
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L126
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L244
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L246

public functions should be placed after all external:
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L41
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L44
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L50

external and public functions should be placed before private:
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L156
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L211
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L221

##### Recommendation
Reorder functions where possible.
#
### NC-3. Typo in event name
##### Instances
- [```event WeightChange(uint indexed index, uint weight);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L28) => ```WeightChanged```

#
### NC-4. Constants may be used
##### Description
Constants may be used instead for repeating values.
##### Instances
10 ** 18:
- [```uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L60) 
- [```return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L87) 
- [```uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L74) 
- [```(10 ** 18 - maxSlippage)) / 10 ** 18;```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L75) 
- [```10 ** 18```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L113) 
- [```return ((10 ** 18 * frxAmount) /```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L115) 
- [```uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173) 
- [```((10 ** 18 - maxSlippage))) / 10 ** 18);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L174) 
- [```RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L214) 
- [```else return (poolPrice() * 10 ** 18) / (10 ** 18);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215) 
- [```10 ** 18;```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L75) 
- [```preDepositPrice = 10 ** 18; // initializes with a price of 1```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L80) 
- [```else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L81) 
- [```) * depositAmount) / 10 ** 18;```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L94) 
- [```uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L98) 

500:
- [```500,```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L180)
- [```factory.getPool(rocketTokenRETHAddress, W_ETH_ADDRESS, 500)```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L238)

##### Recommendation
Define constant variables for repeated values.
#
### NC-5. Make event names more consistent
##### Description
Change to “MinAmountChanged” and “MaxAmountChanged”.
##### Instances
- [```event ChangeMinAmount(uint256 indexed minAmount);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L21) 
- [```event ChangeMaxAmount(uint256 indexed maxAmount);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L22) 

##### Recommendation
Consider renaming events.
#
### NC-6. Emit events in initialize() functions
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L33
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L36
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L42
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L48

##### Recommendation
For off-chain monitoring and transparency, consider emitting an event in initialize() function.
#
### NC-7. Put ```deposit()``` before ```withdraw()```
##### Description
It is unusual and a little confusing that the deposit() function is placed after the withdraw() function.
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L73
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L94
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L156

##### Recommendation
To make the code more well-structured, place the deposit() function before withdraw().
#
### NC-8. Concatenate multiple lines of code
##### Description
This complicates and lengthens the code when several small pieces of code are placed on several lines.
##### Instances
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L64
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L67
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L85
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L97
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L99
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L103
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L113
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L111
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L75
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L125
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L150

##### Recommendation
If you combine the specified lines with the previous ones, then they will not become too long.  
For example, it looks better:
```
(bool sent, ) = address(msg.sender).call{value: address(this).balance}("");
```
than
```
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
```
#
### NC-9. Incorrect comment
##### Description
The comment for the adjustWeight() function says that it "Adds new derivative to the index fund", which is not true.
##### Instances
- [```Link```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L158-L165): 
```
        @notice - Adds new derivative to the index fund
        @dev - Weights are only in regards to each other, total weight changes with this function
        @dev - If you want exact weights either do the math off chain or reset all existing derivates to the weights you want
        @dev - Weights are approximate as it will slowly change as people stake
        @param _derivativeIndex - index of the derivative you want to update the weight
        @param _weight - new weight for this derivative.
    */
    function adjustWeight(
```

##### Recommendation
Change to ```@notice - Sets new weight for a certain derivative index```
#
### NC-10. Make commentss more consistent
##### Description
To make comments for all functions with the onlyOwner modifier more consistent, change the two comments.
##### Instances
- [```@notice - Sets the max slippage for a certain derivative index```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L198) => "Owner only function that sets the max slippage for a certain derivative index"
- [```@notice - Sets the minimum amount a user is allowed to stake```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L211) => "Owner only function that sets the minimum amount a user is allowed to stake"

#
### NC-11. NatSpec is incomplete
##### Description
NatSpec does not have parameter descriptions for some functions.
##### Instances
- [```function setMaxSlippage(uint256 _slippage) external onlyOwner {```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48) add ```_slippage``` description
- [```function withdraw(uint256 _amount) external onlyOwner {```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56) add ```_amount``` description
- [```function ethPerDerivative(uint256 _amount) public view returns (uint256) {```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86) add ```_amount``` description
- [```function setMaxSlippage(uint256 _slippage) external onlyOwner {```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51) add ```_slippage``` description
- [```function ethPerDerivative(uint256 _amount) public view returns (uint256) {```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L111) add ```_amount``` description
- [```function withdraw(uint256 amount) external onlyOwner {```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107) add ```_amount``` description

#
### NC-12. Inconsistency when using uint and uint256
##### Description
Some variables is declared as ```uint``` and some as ```uint256```.
##### Instances
There are 63 instances using ```uint256``` and 16 instances using ```uint```.
- [```event ChangeMinAmount(uint256 indexed minAmount);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L21)
- [```event Staked(address indexed recipient, uint ethIn, uint safEthOut);```](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L26) 

##### Recommendation
Stick to one style. 