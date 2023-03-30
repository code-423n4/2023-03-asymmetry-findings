# Gas Optimization

# Summary

| Number | Optimization Details                                                                | Context |
| :----: | :---------------------------------------------------------------------------------- | :-----: |
| [G-01] | FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE    |   14    |
| [G-02] | PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD     |    8    |
| [G-03] | SETTING THE CONSTRUCTOR TO PAYABLE                                                  |    4    |
| [G-04] | NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS |    1    |
| [G-05] | CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS                                  |    2    |
| [G-06] | The result of function calls should be cached rather than re-calling the function   |    2    |
| [G-07] | Duplicated require() checks should be refactored to a modifier or function          |    1    |
| [G-08] | With assembly, .call (bool success) transfer can be done gas-optimized              |    5    |

## [G-01] FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE

### Summary

The onlyOwner modifier makes a function revert if not called by the address registered as the owner

### Details

There are **14** instances of this issue.

### Github Permalinks

```solidity
File: /SafEth/derivatives/WstEth.sol

48     function setMaxSlippage(uint256 _slippage) external onlyOwner {

56    function withdraw(uint256 _amount) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

```solidity
File: /SafEth/derivatives/SfrxEth.sol

51    function setMaxSlippage(uint256 _slippage) external onlyOwner {

60    function withdraw(uint256 _amount) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

```solidity
File: /SafEth/SafEth.sol

138    function rebalanceToWeights() external onlyOwner {

165    function adjustWeight(
166        uint256 _derivativeIndex,
167        uint256 _weight
168:    ) external onlyOwner {


182   function addDerivative(
183        address _contractAddress,
184        uint256 _weight
185:    ) external onlyOwner {

202 function setMaxSlippage(
203        uint _derivativeIndex,
204        uint _slippage
205:    ) external onlyOwner {

214    function setMinAmount(uint256 _minAmount) external onlyOwner {

223    function setMaxAmount(uint256 _maxAmount) external onlyOwner {

232    function setPauseStaking(bool _pause) external onlyOwner {

241    function setPauseUnstaking(bool _pause) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

```solidity
File: /SafEth/derivatives/Reth.sol

58    function setMaxSlippage(uint256 _slippage) external onlyOwner {

107    function withdraw(uint256 amount) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

## [G-02] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD

### Details

There are **8** instances of this issue.

### Github Permalinks

```solidity
File: /SafEth/derivatives/WstEth.sol

41    function name() public pure returns (string memory) {

86    function ethPerDerivative(uint256 _amount) public view returns (uint256) {

93    function balance() public view returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

```solidity
File: /SafEth/derivatives/SfrxEth.sol

44    function name() public pure returns (string memory) {

122    function balance() public view returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

```solidity
File: /SafEth/derivatives/Reth.sol

50    function name() public pure returns (string memory) {

211    function ethPerDerivative(uint256 _amount) public view returns (uint256) {

221    function balance() public view returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

## [G-03] SETTING THE CONSTRUCTOR TO PAYABLE

### Details

There are **4** instances of this issue.

### Github Permalinks

```solidity
File: /SafEth/derivatives/WstEth.sol

24    constructor() {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L24

```solidity
File: /derivatives/SfrxEth.sol

27      constructor() {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L27

```solidity
File: /SafEth/SafEth.sol

38    constructor() {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L38

```solidity
File: /SafEth/derivatives/Reth.sol

33    constructor() {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L33

## [G-04] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /SafEth/derivatives/Reth.sol

89    ) private returns (uint256 amountOut) {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L89

## [G-05] CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS

### Details

There are **2** instances of this issue.

### Github Permalinks

```solidity
File: /SafEth/SafEth.sol

115            uint256 derivativeAmount = (derivatives[i].balance() *

149            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

## [G-06] The result of function calls should be cached rather than re-calling the function

### Details

There are **2** instances of this issue.

### Github Permalinks

```solidity
File: /SafEth/derivatives/Reth.sol

170        if (!poolCanDeposit(msg.value)) {

212        if (poolCanDeposit(_amount))
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

## [G-07] Duplicated require() checks should be refactored to a modifier or function

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /SafEth/derivatives/WstEth.sol

66         require(sent, "Failed to send Ether");
77         require(sent, "Failed to send Ether");
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

## [G-08] With assembly, .call (bool success) transfer can be done gas-optimized

### Summary

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19

### Details

There are **5** instances of this issue.

### Github Permalinks

```solidity
File: /SafEth/derivatives/SfrxEth.sol

-84:        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
-85     ""
-86        );


+           bool sent;
+            assembly {
+                sent := call(gas(), address(msg.sender), address(this).balance, 0, 0)
+            }
+
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L84

```solidity
File: /SafEth/SafEth.sol

124        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L124

```solidity
File: /SafEth/derivatives/Reth.sol

110        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L110

```solidity
File: /SafEth/derivatives/WstEth.sol

63        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(

76        (bool sent, ) = WST_ETH.call{value: msg.value}("");
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol
