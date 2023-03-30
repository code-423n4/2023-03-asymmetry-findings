## QA
---

### Function Visibility [1]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), public, external, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

```solidity
// receive function should come before all the other functions
246:    receive() external payable {}
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

```solidity
// public functions coming after external ones
44:    function name() public pure returns (string memory) {
111:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
122:    function balance() public view returns (uint256) {

// receive function should come before all the other functions
126:    receive() external payable {}
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

```solidity
// public functions coming after external ones
50:    function name() public pure returns (string memory) {
211:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
221:    function balance() public view returns (uint256) {


// external functions coming after private functions
107:    function withdraw(uint256 amount) external onlyOwner {
156:    function deposit() external payable onlyOwner returns (uint256) {


// receive function should come before all the other functions
244:    receive() external payable {}
```

---

### natSpec missing [2]

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

```solidity
21:    event ChangeMinAmount(uint256 indexed minAmount);
22:    event ChangeMaxAmount(uint256 indexed maxAmount);
23:    event StakingPaused(bool indexed paused);
24:    event UnstakingPaused(bool indexed paused);
25:    event SetMaxSlippage(uint256 indexed index, uint256 slippage);
26:    event Staked(address indexed recipient, uint ethIn, uint safEthOut);
27:    event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);
28:    event WeightChange(uint indexed index, uint weight);
29:    event DerivativeAdded(
34:    event Rebalanced();
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

```solidity
// @return missing
41:    function name() public pure returns (string memory) {
73:    function deposit() external payable onlyOwner returns (uint256) {
93:    function balance() public view returns (uint256) {

// @param missing
48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
56:    function withdraw(uint256 _amount) external onlyOwner {

// @param and @return missing
86:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {

97:    receive() external payable {}
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

```solidity
// @return missing
44:    function name() public pure returns (string memory) {
94:    function deposit() external payable onlyOwner returns (uint256) {
122:    function balance() public view returns (uint256) {

// @param missing
51:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

// @param and @return missing
111:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {

126:    receive() external payable {}
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

```solidity
// @return missing
50:    function name() public pure returns (string memory) {
83:    function swapExactInputSingleHop(

// @return missing
66:    function rethAddress() private view returns (address) {
107:    function withdraw(uint256 amount) external onlyOwner {
120:    function poolCanDeposit(uint256 _amount) private view returns (bool) {
156:    function deposit() external payable onlyOwner returns (uint256) {
211:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
221:    function balance() public view returns (uint256) {
228:    function poolPrice() private view returns (uint256) {


244:    receive() external payable {}
```

---

### State variable and function names [3]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

```solidity
// private and internal `functions` should preppend with `underline`
66:    function rethAddress() private view returns (address) {
83:    function swapExactInputSingleHop(
120:    function poolCanDeposit(uint256 _amount) private view returns (bool) {
228:    function poolPrice() private view returns (uint256) {
```

---

### Version [4]

- Pragma versions should be standardized and avoid floating pragma `( ^ )`.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEthStorage.sol

```solidity
// lose the caret ^ for safer code
2:  pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

```solidity
// lose the caret ^ for safer code
2:  pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

```solidity
// lose the caret ^ for safer code
2:  pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

```solidity
// lose the caret ^ for safer code
2:  pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

```solidity
// lose the caret ^ for safer code
2:  pragma solidity ^0.8.13;
```

---

### Observations [5]

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

```solidity
// The @notice is incorrect as this does add a new derivative but adjusts the weight
158:        @notice - Adds new derivative to the index fund
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEthStorage.sol

```solidity
// little typo here, should be "true if unstaking is paused"
17:    bool public pauseUnstaking; // true if unstaking is pause
```

### Initialized variables with default value [6]

- Variables are default initialized with 0 for `uint / int`, 0x0 for `address` and false for `bool`

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

```solidity
// these are redundant and cost more gas
68:        uint256 underlyingValue = 0;
83:        uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
190:        uint256 localTotalWeight = 0;
```