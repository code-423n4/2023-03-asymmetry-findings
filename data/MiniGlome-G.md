## Gas Optimizations
| |Issue|Instances|
|-|:-|:-:|
| [GAS-01] | The result of function calls should be cached rather than re-calling the function | 2 |
| [GAS-02] | For loop can be replace by simple addition | 1 | 
| [GAS-03] | Check should be done before For loop | 1 | 
| [GAS-04] | Check should be done beforehand | 1 | 
| [GAS-05] | Unnecessary calculations | 1 | 
| [GAS-06] | Functions guaranteed to revert when called by normal users can be marked `payable` | 14 | 
| [GAS-07] | Increments can be `unchecked` | 1 | 
| [GAS-08] | Setting the `constructor` to `payable` | 4 |  

### [GAS-01] The result of function calls should be cached rather than re-calling the function
The instances below point to the second+ call of the function within a single function.

*Instances (2)*:
```solidity
File: contracts/SafEth/SafEth.sol
74:					derivatives[i].balance()

142:				derivatives[i].withdraw(derivatives[i].balance());

```
Consider storing the value of `derivatives[i].balance()` in a local variable instead of calling this function twice.

### [GAS-02] For loop can be replace by simple addition
In the following code, the for loop can be replaced by a simple addition because only the `_weight` is to be added to the `totalWeight`.

*Instance (1)*:
```solidity
File: contracts/SafEth/SafEth.sol
190:	uint256 localTotalWeight = 0;
		for (uint256 i = 0; i < derivativeCount; i++)
			localTotalWeight += weights[i];
		totalWeight = localTotalWeight;

```
Can be replaced by:
```solidity
totalWeight = totalWeight + _weight;
```

### [GAS-03] Check should be done before For loop
The check of `ethAmountToRebalance` should be done before the for loop.

*Instance (1)*:
```solidity
File: contracts/SafEth/SafEth.sol
148:	for (uint i = 0; i < derivativeCount; i++) {
			if (weights[i] == 0 || ethAmountToRebalance == 0) continue; // @audit edit this check
			uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
				totalWeight;
			// Price will change due to slippage
			derivatives[i].deposit{value: ethAmount}();
		}

```
Can be refactored as such:

```solidity
if (ethAmountToRebalance != 0) { // @audit edit here
	for (uint i = 0; i < derivativeCount; i++) {
		if (weights[i] == 0) continue; // @audit edit here
		uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
			totalWeight;
		// Price will change due to slippage
		derivatives[i].deposit{value: ethAmount}();
	}
}

```

### [GAS-04] Check should be done beforehand
Value check of `weight` could be done just after the initialization of the variable.

*Instance (1)*:
```solidity
File: contracts/SafEth/SafEth.sol
85:		uint256 weight = weights[i];
86:		IDerivative derivative = derivatives[i];
87:		if (weight == 0) continue; // @audit check should be done beforehand

```
Line 86 and 87 should be swapped.

### [GAS-05] Unnecessary calculations
Multiplication then division by the same number (10 ** 18) is unnecessary as it does not change the initial value.

*Instance (1)*:
```solidity
File: contracts/SafEth/derivatives/Reth.sol
215:	else return (poolPrice() * 10 ** 18) / (10 ** 18);	

```
This line can be replaced by:
```solidity
	else return poolPrice();	

```


### [GAS-06] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are `CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

*Instances (14)*:
```solidity
File: contracts/SafEth/SafEth.sol
138:	function rebalanceToWeights() external onlyOwner {

214:	function setMinAmount(uint256 _minAmount) external onlyOwner {

223:	function setMaxAmount(uint256 _maxAmount) external onlyOwner {

232:	function setPauseStaking(bool _pause) external onlyOwner {

241:	function setPauseUnstaking(bool _pause) external onlyOwner {

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol
58:	function setMaxSlippage(uint256 _slippage) external onlyOwner {

107:	function withdraw(uint256 amount) external onlyOwner {

156:	function deposit() external payable onlyOwner returns (uint256) {

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol
51:	function setMaxSlippage(uint256 _slippage) external onlyOwner {

60:	function withdraw(uint256 _amount) external onlyOwner {

94:	function deposit() external payable onlyOwner returns (uint256) {

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol
48:	function setMaxSlippage(uint256 _slippage) external onlyOwner {

56:	function withdraw(uint256 _amount) external onlyOwner {

73:	function deposit() external payable onlyOwner returns (uint256) {

```

### [GAS-07] Increments can be `unchecked`
Increments for uint256 iterators cannot realistically overflow as this would require too many increments, so this can be `unchecked`.
		The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP.

*Instances (1)*:
```solidity
File: contracts/SafEth/SafEth.sol
188:		derivativeCount++;

```

### [GAS-08] Setting the `constructor` to `payable`
Saves ~13 gas per instance

*Instances (4)*:
```solidity
File: contracts/SafEth/SafEth.sol
38:	constructor() {

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol
33:	constructor() {

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol
27:	constructor() {

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol
24:	constructor() {

```
