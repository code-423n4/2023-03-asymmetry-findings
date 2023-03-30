# Gas Optimizations list

| Number | Details                                                                                           | Instances |
| ------ | ------------------------------------------------------------------------------------------------- | --------- |
| 1      | ```x += y```/```x -= y``` COSTS MORE GAS THAN ```x = x + y```/```x = x - y``` FOR STATE VARIABLES | 3         |
| 2      | CAN DECLARE VARIABLE OUTSIDE LOOP TO SAVE GAS                                                     | 7         |
| 3      | PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD                   | 8         |
| 4      | USE ```bytes32``` INSTEAD OF ```string``` WHEREVER POSSIBLE                                       | 1         |
| 5      | OPTIMIZE NAMES TO SAVE GAS                                                                        | 4         |
| 6      | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE                       | 9         |
| 7      | CACHE STORAGE VALUES IN MEMORY TO MINIMIZE SLOADS                                                 | 8         |
| 8      | SHOULD USE LOCAL ARGUMENTS INSTEAD OF STATE VARIABLE                                              | 4         |
| 9      | CACHE ARRAY LENGTH OUTSIDE OF LOOP                                                                | 7         |

# Gas Optimizations
## [G-01]  ```x += y```/```x -= y``` COSTS MORE GAS THAN ```x = x + y```/```x = x - y``` FOR STATE VARIABLES
Using the addition operator instead of plus-equals saves some gas for state variables.

[contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)
```js
72:             underlyingValue +=
73:                 (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74:                     derivatives[i].balance()) /
75:                 10 ** 18;

172:             localTotalWeight += weights[i];

192:             localTotalWeight += weights[i];
```


## [G-02] CAN DECLARE VARIABLE OUTSIDE LOOP TO SAVE GAS 
Declaring the stack variable outside the loop will save gas that would otherwise be spent on declaring the variable over and over again.     

[contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)
```js
85:             uint256 weight = weights[i];

86:             IDerivative derivative = derivatives[i];

88:             uint256 ethAmount = (msg.value * weight) / totalWeight;

91:             uint256 depositAmount = derivative.deposit{value: ethAmount}();
92:             uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
93:                 depositAmount
94:             ) * depositAmount) / 10 ** 18;
95:             totalStakeValueEth += derivativeReceivedEthValue;

115:             uint256 derivativeAmount = (derivatives[i].balance() *

149:             uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
```


## [G-03] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD
Contracts are allowed to override their parents’ functions and change the visibility from public to external and can save gas by doing so.

[contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol)
```js
50:     function name() public pure returns (string memory) {

211:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {

221:     function balance() public view returns (uint256) {
```

[contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol)
```js
44:     function name() public pure returns (string memory) {

122:     function balance() public view returns (uint256) {
```

[contracts/SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol)
```js
41:     function name() public pure returns (string memory) {

86:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {

93:     function balance() public view returns (uint256) {
```


## [G-04] USE ```bytes32``` INSTEAD OF ```string``` WHEREVER POSSIBLE
Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

[contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol)
```js
50:     function name() public pure returns (string memory) {
```

[contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol)
```js
44:     function name() public pure returns (string memory) {
```

[contracts/SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol)
```js
41:     function name() public pure returns (string memory) {
```

## [G-05] OPTIMIZE NAMES TO SAVE GAS
Function hashes that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per sorted position shifted. Prioritize the most called functions and sort and rename them according to the function hashes/method IDs. For a better understanding please refer to [this link](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

The method IDs in the SafEth.sol will be used the most. A lower method ID may be given to the most frequently used functions. [This](https://emn178.github.io/solidity-optimize-name/) is a useful tool that can be used for the same, if required.

```json
Sighash   |   Function Signature
========================
4cd88b76  =>  initialize(string,string)
3a4b66f1  =>  stake()
2e17de78  =>  unstake(uint256)
d416c096  =>  rebalanceToWeights()
b9c849b6  =>  adjustWeight(uint256,uint256)
a57e3b10  =>  addDerivative(address,uint256)
5e7cc583  =>  setMaxSlippage(uint256,uint256)
897b0637  =>  setMinAmount(uint256)
4fe47f70  =>  setMaxAmount(uint256)
6d49e0b6  =>  setPauseStaking(bool)
d5b7f606  =>  setPauseUnstaking(bool)
```

## [G-06] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE
Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything.

[contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)
```js
108:     function unstake(uint256 _safEthAmount) external {
```

[contracts/SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol)
```js
73:     function deposit() external payable onlyOwner returns (uint256) {

56:     function withdraw(uint256 _amount) external onlyOwner {
```

[contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol)
```js
60:     function withdraw(uint256 _amount) external onlyOwner {

94:     function deposit() external payable onlyOwner returns (uint256) {

111:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

[contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol)
```js
107:     function withdraw(uint256 amount) external onlyOwner {

156:     function deposit() external payable onlyOwner returns (uint256) {

211:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

## [G-07] CACHE STORAGE VALUES IN MEMORY TO MINIMIZE SLOADS
SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

[contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)
```js
71:         for (uint i = 0; i < derivativeCount; i++)

84:         for (uint i = 0; i < derivativeCount; i++) {

113:         for (uint256 i = 0; i < derivativeCount; i++) {

140:         for (uint i = 0; i < derivativeCount; i++) {

141:             if (derivatives[i].balance() > 0)

148:             if (weights[i] == 0 || ethAmountToRebalance == 0) continue;

147:         for (uint i = 0; i < derivativeCount; i++) {

186:         derivatives[derivativeCount] = IDerivative(_contractAddress);
```

## [G-08] SHOULD USE LOCAL ARGUMENTS INSTEAD OF STATE VARIABLE
Using the local arguments of the function while emitting an event instead of the state variable saves roughly 122 gas per emit.

[contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)
```js
216:         emit ChangeMinAmount(minAmount);

225:         emit ChangeMaxAmount(maxAmount);

234:         emit StakingPaused(pauseStaking);

243:         emit UnstakingPaused(pauseUnstaking);
```

## [G-09] CACHE ARRAY LENGTH OUTSIDE OF LOOP
Caching the array length outside a loop saves reading it on each iteration, as long as the array’s length is not changed during the loop.

[contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)
```js
71:         for (uint i = 0; i < derivativeCount; i++)

84:         for (uint i = 0; i < derivativeCount; i++) {

113:         for (uint256 i = 0; i < derivativeCount; i++) {

140:         for (uint i = 0; i < derivativeCount; i++) {

147:         for (uint i = 0; i < derivativeCount; i++) {

171:         for (uint256 i = 0; i < derivativeCount; i++)

191:         for (uint256 i = 0; i < derivativeCount; i++)
```