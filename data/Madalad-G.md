# Gas Optimizations Summary
| |Issue|Instances|
|:-:|:-|:-:|
|[G-01]|Functions guaranteed to revert when called by normal users can be marked `payable`|14|
|[G-02]|Use assembly to calculate hashes|6|
|[G-03]|Do not compare boolean expressions to boolean literals|2|
|[G-04]|Division/Multiplication by 2 should use bit shifting|1|
|[G-05]|Use `indexed` to save gas|6|
|[G-06]|Use `unchecked` for operations that cannot overflow/underflow|7|
|[G-07]|Use `private` rather than `public` for constants|11|
|[G-08]|Change `public` functions to `external`|8|
|[G-09]|Use named return values|15|

Total issues: 9

Total instances: 70

&nbsp;
# Gas Optimizations
## [G-01] Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost (2400 per instance).

Instances: 14
```solidity
File: contracts/SafEth/derivatives/Reth.sol

58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

107:     function withdraw(uint256 amount) external onlyOwner {

```
- [contracts/SafEth/derivatives/Reth.sol#L58](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58)
- [contracts/SafEth/derivatives/Reth.sol#L107](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107)

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

48:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

56:     function withdraw(uint256 _amount) external onlyOwner {

```
- [contracts/SafEth/derivatives/WstEth.sol#L48](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48)
- [contracts/SafEth/derivatives/WstEth.sol#L56](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56)

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

51:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

60:     function withdraw(uint256 _amount) external onlyOwner {

```
- [contracts/SafEth/derivatives/SfrxEth.sol#L51](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51)
- [contracts/SafEth/derivatives/SfrxEth.sol#L60](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60)

```solidity
File: contracts/SafEth//SafEth.sol

138:     function rebalanceToWeights() external onlyOwner {

165:     function adjustWeight(

182:     function addDerivative(

202:     function setMaxSlippage(

214:     function setMinAmount(uint256 _minAmount) external onlyOwner {

223:     function setMaxAmount(uint256 _maxAmount) external onlyOwner {

232:     function setPauseStaking(bool _pause) external onlyOwner {

241:     function setPauseUnstaking(bool _pause) external onlyOwner {

```
- [contracts/SafEth//SafEth.sol#L138](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L138)
- [contracts/SafEth//SafEth.sol#L165](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L165)
- [contracts/SafEth//SafEth.sol#L182](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L182)
- [contracts/SafEth//SafEth.sol#L202](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L202)
- [contracts/SafEth//SafEth.sol#L214](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L214)
- [contracts/SafEth//SafEth.sol#L223](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L223)
- [contracts/SafEth//SafEth.sol#L232](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L232)
- [contracts/SafEth//SafEth.sol#L241](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L241)


&nbsp;
## [G-02] Use assembly to calculate hashes

Saves 5000 deployment gas per instance and 80 runtime gas per instance.

### Unoptimized
```solidity
function solidityHash(uint256 a, uint256 b) public view {
	//unoptimized
	keccak256(abi.encodePacked(a, b));
}
```

### Optimized
```solidity
function assemblyHash(uint256 a, uint256 b) public view {
	//optimized
	assembly {
		mstore(0x00, a)
		mstore(0x20, b)
		let hashedVal := keccak256(0x00, 0x40)
	}
}
```

Instances: 6
```solidity
File: contracts/SafEth/derivatives/Reth.sol

69:                 keccak256(

124:                 keccak256(

135:                 keccak256(

161:                 keccak256(

190:                     keccak256(

232:                 keccak256(

```
- [contracts/SafEth/derivatives/Reth.sol#L69](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L69)
- [contracts/SafEth/derivatives/Reth.sol#L124](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L124)
- [contracts/SafEth/derivatives/Reth.sol#L135](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L135)
- [contracts/SafEth/derivatives/Reth.sol#L161](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L161)
- [contracts/SafEth/derivatives/Reth.sol#L190](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L190)
- [contracts/SafEth/derivatives/Reth.sol#L232](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L232)


&nbsp;
## [G-03] Do not compare boolean expressions to boolean literals

`<x> == true` <=> `<x>`, also `<x> == false` <=> `!<x>`

Instances: 2
```solidity
File: contracts/SafEth//SafEth.sol

64:         require(pauseStaking == false, "staking is paused");

109:         require(pauseUnstaking == false, "unstaking is paused");

```
- [contracts/SafEth//SafEth.sol#L64](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L64)
- [contracts/SafEth//SafEth.sol#L109](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L109)


&nbsp;
## [G-04] Division/Multiplication by 2 should use bit shifting

`<x> * 2` is equivalent to `<x> << 1` and `<x> / 2` is the same as `<x> >> 1`. The `MUL` and `DIV` opcodes cost 5 gas, whereas `SHL` and `SHR` only cost 3 gas.

Instances: 1
```solidity
File: contracts/SafEth/derivatives/Reth.sol

241:         return (sqrtPriceX96 * (uint(sqrtPriceX96)) * (1e18)) >> (96 * 2);

```
- [contracts/SafEth/derivatives/Reth.sol#L241](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L241)


&nbsp;
## [G-05] Use `indexed` to save gas
Using `indexed` for value type event parameters (address, uint, bool) saves at least 80 gas in emitting the event (https://gist.github.com/Tomosuke0930/9bf61e01a8c3e214d95a9b84dcb41d97).

Note that for other types however (string, bytes), it is more expensive.

Instances: 6
```solidity
File: contracts/SafEth//SafEth.sol

25:     event SetMaxSlippage(uint256 indexed index, uint256 slippage);

26:     event Staked(address indexed recipient, uint ethIn, uint safEthOut);

27:     event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);

28:     event WeightChange(uint indexed index, uint weight);

31:         uint weight,

32:         uint index

```
- [contracts/SafEth//SafEth.sol#L25](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L25)
- [contracts/SafEth//SafEth.sol#L26](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L26)
- [contracts/SafEth//SafEth.sol#L27](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L27)
- [contracts/SafEth//SafEth.sol#L28](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L28)
- [contracts/SafEth//SafEth.sol#L31](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L31)
- [contracts/SafEth//SafEth.sol#L32](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L32)


&nbsp;
## [G-06] Use `unchecked` for operations that cannot overflow/underflow

By bypassing Solidity's built in overflow/underflow checks using `unchecked`, we can save gas. This is especially beneficial for the index variable within for loops (saves 120 gas per iteration).

Instances: 7
```solidity
File: contracts/SafEth//SafEth.sol

71:         for (uint i = 0; i < derivativeCount; i++)

84:         for (uint i = 0; i < derivativeCount; i++) {

113:         for (uint256 i = 0; i < derivativeCount; i++) {

140:         for (uint i = 0; i < derivativeCount; i++) {

147:         for (uint i = 0; i < derivativeCount; i++) {

171:         for (uint256 i = 0; i < derivativeCount; i++)

191:         for (uint256 i = 0; i < derivativeCount; i++)

```
- [contracts/SafEth//SafEth.sol#L71](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L71)
- [contracts/SafEth//SafEth.sol#L84](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L84)
- [contracts/SafEth//SafEth.sol#L113](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L113)
- [contracts/SafEth//SafEth.sol#L140](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L140)
- [contracts/SafEth//SafEth.sol#L147](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L147)
- [contracts/SafEth//SafEth.sol#L171](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L171)
- [contracts/SafEth//SafEth.sol#L191](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth//SafEth.sol#L191)


&nbsp;
## [G-07] Use `private` rather than `public` for constants

Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table. If needed to be viewed externally, the values can be read from the verified contract source code.

Instances: 11
```solidity
File: contracts/SafEth/derivatives/Reth.sol

20:     address public constant ROCKET_STORAGE_ADDRESS =

22:     address public constant W_ETH_ADDRESS =

24:     address public constant UNISWAP_ROUTER =

26:     address public constant UNI_V3_FACTORY =

```
- [contracts/SafEth/derivatives/Reth.sol#L20](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L20)
- [contracts/SafEth/derivatives/Reth.sol#L22](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L22)
- [contracts/SafEth/derivatives/Reth.sol#L24](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L24)
- [contracts/SafEth/derivatives/Reth.sol#L26](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L26)

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

13:     address public constant WST_ETH =

15:     address public constant LIDO_CRV_POOL =

17:     address public constant STETH_TOKEN =

```
- [contracts/SafEth/derivatives/WstEth.sol#L13](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L13)
- [contracts/SafEth/derivatives/WstEth.sol#L15](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L15)
- [contracts/SafEth/derivatives/WstEth.sol#L17](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L17)

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

14:     address public constant SFRX_ETH_ADDRESS =

16:     address public constant FRX_ETH_ADDRESS =

18:     address public constant FRX_ETH_CRV_POOL_ADDRESS =

20:     address public constant FRX_ETH_MINTER_ADDRESS =

```
- [contracts/SafEth/derivatives/SfrxEth.sol#L14](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L14)
- [contracts/SafEth/derivatives/SfrxEth.sol#L16](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L16)
- [contracts/SafEth/derivatives/SfrxEth.sol#L18](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L18)
- [contracts/SafEth/derivatives/SfrxEth.sol#L20](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L20)


&nbsp;
## [G-08] Change `public` functions to `external`

Functions marked as `public` that are not called internally should be set to `external` to save gas and improve code quality. External call cost is less expensive than of public functions.

Instances: 8
```solidity
File: contracts/SafEth/derivatives/Reth.sol

50:     function name() public pure returns (string memory) {

211:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {

221:     function balance() public view returns (uint256) {

```
- [contracts/SafEth/derivatives/Reth.sol#L50](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L50)
- [contracts/SafEth/derivatives/Reth.sol#L211](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L211)
- [contracts/SafEth/derivatives/Reth.sol#L221](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L221)

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

41:     function name() public pure returns (string memory) {

86:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {

93:     function balance() public view returns (uint256) {

```
- [contracts/SafEth/derivatives/WstEth.sol#L41](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L41)
- [contracts/SafEth/derivatives/WstEth.sol#L86](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86)
- [contracts/SafEth/derivatives/WstEth.sol#L93](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L93)

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

44:     function name() public pure returns (string memory) {

122:     function balance() public view returns (uint256) {

```
- [contracts/SafEth/derivatives/SfrxEth.sol#L44](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L44)
- [contracts/SafEth/derivatives/SfrxEth.sol#L122](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L122)


&nbsp;
## [G-09] Use named return values

Using named return values instead of explicitly calling `return` saves gas.

Instances: 15
```solidity
File: contracts/SafEth/derivatives/Reth.sol

50:     function name() public pure returns (string memory) {

66:     function rethAddress() private view returns (address) {

120:     function poolCanDeposit(uint256 _amount) private view returns (bool) {

156:     function deposit() external payable onlyOwner returns (uint256) {

211:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {

221:     function balance() public view returns (uint256) {

228:     function poolPrice() private view returns (uint256) {

```
- [contracts/SafEth/derivatives/Reth.sol#L50](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L50)
- [contracts/SafEth/derivatives/Reth.sol#L66](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L66)
- [contracts/SafEth/derivatives/Reth.sol#L120](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L120)
- [contracts/SafEth/derivatives/Reth.sol#L156](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L156)
- [contracts/SafEth/derivatives/Reth.sol#L211](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L211)
- [contracts/SafEth/derivatives/Reth.sol#L221](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L221)
- [contracts/SafEth/derivatives/Reth.sol#L228](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L228)

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

41:     function name() public pure returns (string memory) {

73:     function deposit() external payable onlyOwner returns (uint256) {

86:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {

93:     function balance() public view returns (uint256) {

```
- [contracts/SafEth/derivatives/WstEth.sol#L41](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L41)
- [contracts/SafEth/derivatives/WstEth.sol#L73](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L73)
- [contracts/SafEth/derivatives/WstEth.sol#L86](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86)
- [contracts/SafEth/derivatives/WstEth.sol#L93](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L93)

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

44:     function name() public pure returns (string memory) {

94:     function deposit() external payable onlyOwner returns (uint256) {

111:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {

122:     function balance() public view returns (uint256) {

```
- [contracts/SafEth/derivatives/SfrxEth.sol#L44](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L44)
- [contracts/SafEth/derivatives/SfrxEth.sol#L94](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L94)
- [contracts/SafEth/derivatives/SfrxEth.sol#L111](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L111)
- [contracts/SafEth/derivatives/SfrxEth.sol#L122](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L122)


&nbsp;
