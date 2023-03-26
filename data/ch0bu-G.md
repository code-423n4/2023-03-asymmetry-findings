## 1. Using unchecked blocks to save gas - increments in for loop can be unchecked

The majority of Solidity for loops increment a uint256 variable that starts at 0. These increment operations never need to be checked for overflow/underflow because the variable will never reach the max number of uint256 (will run out of gas long before that happens). The default overflow/underflow check wastes gas in every iteration of virtually every for loop.

This saves 30-40 gas per loop iteration.

e.g Let’s work with a sample loop below.
```
for(uint256 i; i < 10; i++){
//doSomething
}
```
can be written as shown below.
```
for(uint256 i; i < 10;) {
  // loop logic
  unchecked { i++; }
}
```


```
71       for (uint i = 0; i < derivativeCount; i++)
84       for (uint i = 0; i < derivativeCount; i++) {
113      for (uint256 i = 0; i < derivativeCount; i++) {
140      for (uint i = 0; i < derivativeCount; i++) {
147      for (uint i = 0; i < derivativeCount; i++) {
171      for (uint256 i = 0; i < derivativeCount; i++)
191      for (uint256 i = 0; i < derivativeCount; i++)
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol



## 2. Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. 
- If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified `(if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})`. 
- Empty `receive()`/`fallback()` payable functions that are not used, can be removed to save deployment gas.

```
244      receive() external payable {}
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol
```
246      receive() external payable {}
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol
```
126      receive() external payable {}
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol
```
97       receive() external payable {}
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol


## 3. Don’t compare boolean expressions to boolean literals

`if (<x> == true) => if (<x>), if (<x> == false) => if (!<x>)`


```
64       require(pauseStaking == false, "staking is paused");
109      require(pauseUnstaking == false, "unstaking is paused");
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol


## 4. `10**X` can be changed to `1eX` to save gas

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






