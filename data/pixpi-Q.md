| ID            | Issue                                       | Instances |
| ------------- | :------------------------------------------ | :-------: |
| [QA-1](#QA-1) | Remove unused imports                       |     6     |
| [QA-2](#QA-2) | Specify a fixed `pragma` version            |     5     |
| [QA-3](#QA-3) | Inconsistent use of `uint` / `uint256` type |    16     |
| [QA-4](#QA-4) | Avoid using spot price                      |     1     |
| [QA-5](#QA-5) | Missing NatSpec `@return` comments          |    16     |
| [QA-6](#QA-6) | Missing NatSpec `@param` comments           |     6     |
| [QA-7](#QA-7) | Unused function parameter `_amount`         |     2     |

---

## <a name="QA-1">[QA-1]</a> Remove unused imports

There are several instances of unused `import` statements. Removing them will make the code cleaner and reduce deployment costs.

6 instances - 2 files

```solidity
File: contracts/SafEth/SafEth.sol
  4: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
  5: import "../interfaces/IWETH.sol";
  6: import "../interfaces/uniswap/ISwapRouter.sol";
  7: import "../interfaces/lido/IWStETH.sol";
  8: import "../interfaces/lido/IstETH.sol";
```

```solidity
File: contracts/SafEth/derivatives/Reth.sol
  5: import "../../interfaces/frax/IsFrxEth.sol";
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L4-L8

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L5

---

## <a name="QA-2">[QA-2]</a> Specify a fixed `pragma` version

By locking the pragma version, developers can ensure that their code will work as expected with a known version of the Solidity compiler. This can make it easier to maintain and upgrade Solidity code over time, as new versions of the language are released.

https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/

5 instances - 5 files

```solidity
File: contracts/SafEth/SafEth.sol
  2: pragma solidity ^0.8.13;
```

```solidity
File: contracts/SafEth/SafEthStorage.sol
  2: pragma solidity ^0.8.13;
```

```solidity
File: contracts/SafEth/derivatives/Reth.sol
  2: pragma solidity ^0.8.13;
```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol
  2: pragma solidity ^0.8.13;
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol
  2: pragma solidity ^0.8.13;
```

Recommendation code example:

```solidity
File: contracts/SafEth/SafEth.sol
-  2: pragma solidity ^0.8.13;

+  2: pragma solidity 0.8.13;
```

---

## <a name="QA-3">[QA-3]</a> Inconsistent use of `uint` / `uint256` type

Although `uint` is alias for `uint256` type, using both of them in the project makes code less readable.

Therefore, it is advisable to stick with just one, either `uint` or `uint256`.

16 instances - 2 files

```solidity
File: contracts/SafEth/SafEth.sol
  26:    event Staked(address indexed recipient, uint ethIn, uint safEthOut);

  27:    event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);

  28:    event WeightChange(uint indexed index, uint weight);

  31:        uint weight,

  32:        uint index

  71:        for (uint i = 0; i < derivativeCount; i++)

  84:        for (uint i = 0; i < derivativeCount; i++) {

  92:            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(

  140:        for (uint i = 0; i < derivativeCount; i++) {

  147:        for (uint i = 0; i < derivativeCount; i++) {

  203:        uint _derivativeIndex,

  204:        uint _slippage
```

```solidity
File: contracts/SafEth/derivatives/Reth.sol
  171:            uint rethPerEth = (10 ** 36) / poolPrice();
```

Recommendation:

Replace all `uint` instances with `uint256`.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L26

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L171

---

## <a name="QA-4">[QA-4]</a> Avoid using spot price

In the current implementation of the derivative contract for rETH, the `deposit()` function uses the spot price from Uniswap to calculate the amount of rETH to deposit if `poolCanDeposit(msg.value) == false`.

This introduces some possible risks, as flash loans can be used to manipulate the spot price and drain the smart contract's funds.

Although no critical issues were identified by manipulating the spot price in the current version of the smart contracts, there is a risk that it could be used as an attack vector in future versions.

To mitigate these risks, one possible solution is to use the time-weighted average price (TWAP) from Uniswap instead.

1 instance - 1 file

```solidity
File: contracts/SafEth/derivatives/Reth.sol
  237:        IUniswapV3Pool pool = IUniswapV3Pool(
  238:            factory.getPool(rocketTokenRETHAddress, W_ETH_ADDRESS, 500)
  239:        );
  240:        (uint160 sqrtPriceX96, , , , , , ) = pool.slot0();
  241:        return (sqrtPriceX96 * (uint(sqrtPriceX96)) * (1e18)) >> (96 * 2);
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L237-L241

---

## <a name="QA-5">[QA-5]</a> Missing NatSpec `@return` comments

The provided functions below lack NatSpec `@return` comments, and it is recommended to enhance their documentation by adding them.

16 instances - 3 files

```solidity
File: contracts/SafEth/derivatives/Reth.sol
  50:    function name() public pure returns (string memory) {

  66:    function rethAddress() private view returns (address) {

  89:    ) private returns (uint256 amountOut) {

  120:   function poolCanDeposit(uint256 _amount) private view returns (bool) {

  156:   function deposit() external payable onlyOwner returns (uint256) {

  211:   function ethPerDerivative(uint256 _amount) public view returns (uint256) {

  221:   function balance() public view returns (uint256) {

  228:   function poolPrice() private view returns (uint256) {
```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol
  44:    function name() public pure returns (string memory) {

  94:    function deposit() external payable onlyOwner returns (uint256) {

  111:   function ethPerDerivative(uint256 _amount) public view returns (uint256) {

  122:   function balance() public view returns (uint256) {
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol
  41:    function name() public pure returns (string memory) {

  73:    function deposit() external payable onlyOwner returns (uint256) {

  86:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {

  93:    function balance() public view returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L50

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L44

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L41

---

## <a name="QA-6">[QA-6]</a> Missing NatSpec `@param` comments

The provided functions below lack NatSpec `@param` comments, and it is recommended to enhance their documentation by adding them.

6 instances - 3 files

```solidity
File: contracts/SafEth/derivatives/Reth.sol
  107:    function withdraw(uint256 amount) external onlyOwner {
```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol
  51:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

  111:   function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol
  48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

  56:    function withdraw(uint256 _amount) external onlyOwner {

  86:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L107

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48

---

## Fix imports

---

## <a name="QA-7">[QA-7]</a> Unused function parameter `_amount`

It is recommended to comment out unused parameters in function declarations. That improves code readability and helps to avoid possible mistakes in the future.

2 instances - 2 files

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol
  111:   function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol
  86:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

Recommendation code example:

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol
-  111:   function ethPerDerivative(uint256 _amount) public view returns (uint256) {

+  111:   function ethPerDerivative(uint256 /*_amount*/) public view returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L111

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L86

---
