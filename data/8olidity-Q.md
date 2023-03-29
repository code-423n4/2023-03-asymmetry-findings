## The maxSlippage should be limited

The administrator can set the value of `maxSlippage` through the `setMaxSlippage()` function, but there is no validation here. If `maxSlippage` is greater than `10 ** 18`, it will cause an overflow and make it impossible to  `withdraw()`. This problem exists in several contracts.

```
uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;

```

```
contracts/SafEth/derivatives/Reth.sol:
  57      */
  58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {
  59          maxSlippage = _slippage;

contracts/SafEth/derivatives/SfrxEth.sol:
  50      */
  51:     function setMaxSlippage(uint256 _slippage) external onlyOwner {
  52          maxSlippage = _slippage;

contracts/SafEth/derivatives/WstEth.sol:
  47      */
  48:     function setMaxSlippage(uint256 _slippage) external onlyOwner {
  49          maxSlippage = _slippage;

```

Suggestion

```
function setMaxSlippage(uint256 _slippage) external onlyOwner {
    require(_slippage <= 10 ** 18);
    maxSlippage = _slippage;
}

```

## Code and comments do not match

In the `safeETH::adjustWeight()` function, the function should update the value corresponding to `_derivativeIndex` in the weights array and calculate the new `totalWeight` value.

However, the comment for the function is

```
/**
  @notice - Adds new derivative to the index fund
	.....
*/

```

This comment is the same as the comment in the `addDerivative()` function.

Suggestion

Revise the comment.