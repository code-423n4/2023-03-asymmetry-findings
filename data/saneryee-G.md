# Gas Optimizations Report
| |Issue|Instances|
|-|:-|:-:|
|[G-001]|x += y or x -= y costs more gas than x = x + y or x = x - y for state variables|4|
|[G-002]|Functions guaranteed to revert when called by normal users can be marked payable|14|
|[G-003]|Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate|1|

## [G-001] x += y or x -= y costs more gas than x = x + y or x = x - y for state variables

### Impact
Using the addition operator instead of plus-equals saves 113 gas. Usually does not work with struct and mappings.

### Findings
Total:4

[contracts/SafEth/SafEth.sol#L72](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/SafEth.sol#L72)
```solidity
72:    underlyingValue +=
          (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
              derivatives[i].balance()) /
          10 ** 18;
```
[contracts/SafEth/SafEth.sol#L172](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/SafEth.sol#L172)
```solidity
172:    localTotalWeight += weights[i];
```
[contracts/SafEth/SafEth.sol#L192](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/SafEth.sol#L192)
```solidity
192:    localTotalWeight += weights[i];
```
[contracts/SafEth/SafEth.sol#L95](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/SafEth.sol#L95)
```solidity
95:    totalStakeValueEth += derivativeReceivedEthValue;
```


## [G-002] Functions guaranteed to revert when called by normal users can be marked payable

### Impact
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
   The extra opcodes avoided are `CALLVALUE(2)`,`DUP1(3)`,`ISZERO(3)`,`PUSH2(3)`,`JUMPI(10)`,`PUSH1(3)`,`DUP1(3)`,`REVERT(0)`,`JUMPDEST(1)`,`POP(2)`, which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### Findings
Total:14

[contracts/SafEth/derivatives/WstEth.sol#L56](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/derivatives/WstEth.sol#L56)
```solidity
56:    function withdraw(uint256 _amount) external onlyOwner {
```
[contracts/SafEth/derivatives/WstEth.sol#L48](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/derivatives/WstEth.sol#L48)
```solidity
48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
```
[contracts/SafEth/derivatives/SfrxEth.sol#L60](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/derivatives/SfrxEth.sol#L60)
```solidity
60:    function withdraw(uint256 _amount) external onlyOwner {
```
[contracts/SafEth/derivatives/SfrxEth.sol#L51](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/derivatives/SfrxEth.sol#L51)
```solidity
51:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
```
[contracts/SafEth/SafEth.sol#L168](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/SafEth.sol#L168)
```solidity
168:    ) external onlyOwner {
```
[contracts/SafEth/SafEth.sol#L185](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/SafEth.sol#L185)
```solidity
185:    ) external onlyOwner {
```
[contracts/SafEth/SafEth.sol#L205](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/SafEth.sol#L205)
```solidity
205:    ) external onlyOwner {
```
[contracts/SafEth/SafEth.sol#L214](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/SafEth.sol#L214)
```solidity
214:    function setMinAmount(uint256 _minAmount) external onlyOwner {
```
[contracts/SafEth/SafEth.sol#L232](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/SafEth.sol#L232)
```solidity
232:    function setPauseStaking(bool _pause) external onlyOwner {
```
[contracts/SafEth/SafEth.sol#L223](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/SafEth.sol#L223)
```solidity
223:    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
```
[contracts/SafEth/SafEth.sol#L138](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/SafEth.sol#L138)
```solidity
138:    function rebalanceToWeights() external onlyOwner {
```
[contracts/SafEth/SafEth.sol#L241](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/SafEth.sol#L241)
```solidity
241:    function setPauseUnstaking(bool _pause) external onlyOwner {
```
[contracts/SafEth/derivatives/Reth.sol#L58](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/derivatives/Reth.sol#L58)
```solidity
58:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
```
[contracts/SafEth/derivatives/Reth.sol#L107](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/derivatives/Reth.sol#L107)
```solidity
107:    function withdraw(uint256 amount) external onlyOwner {
```

### Recommendation
Mark the function as payable.

## [G-003] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

### Impact
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

### Findings
Total:1

[contracts/SafEth/SafEthStorage.sol#L22-L23](https://github.com/code-423n4/2023-03-asymmetry/tree/main//contracts/SafEth/SafEthStorage.sol#L22-L23)
```solidity
22:    mapping(uint256 => IDerivative) public derivatives; // derivatives in the system
23:        mapping(uint256 => uint256) public weights; // weights for each derivative
```



