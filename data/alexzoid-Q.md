## [01] Floating Pragma

The compiler version is not locked, which may lead to deploying contracts with different or bugged versions:

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L2
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L2
```solidity
pragma solidity ^0.8.13;
```
To avoid potential issues, lock the pragma version and ensure that the chosen compiler version is free of known bugs:
```solidity
pragma solidity 0.8.19;
```

## [02] Unused Import

An import statement is present in the source file, but the imported file is not used:

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L4-L8
```solidity
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../interfaces/IWETH.sol";
import "../interfaces/uniswap/ISwapRouter.sol";
import "../interfaces/lido/IWStETH.sol";
import "../interfaces/lido/IstETH.sol";
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L5-L6
```solidity
import "../../interfaces/frax/IsFrxEth.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L6
```solidity
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```

To optimize the code and avoid redundancy, remove the unnecessary import.

## [03] Redundant Contract Inheritance

The `Initializable` contract is already inherited through the parent contract `OwnableUpgradeable`:
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L16
```solidity
contract SafEth is
    Initializable,
    ERC20Upgradeable,
    OwnableUpgradeable,
    SafEthStorage
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L19
```solidity
contract Reth is IDerivative, Initializable, OwnableUpgradeable {
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L13
```solidity
contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L12
```solidity
contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
```

Remove the unnecessary extra inheritance of `Initializable`.

## [04] Inappropriate Functions Names

In `initialize()` function consider using more suitable function names at https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L52-L53.
- replace `ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);` with `__ERC20_init(_tokenName, _tokenSymbol);`
- replace `_transferOwnership(msg.sender);` with `__Ownable_init();`

In `initialize()` function replace `_transferOwnership()` with `transferOwnership()`, this will add an important zero address check:
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L43
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L37
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L34

The `stEthPerToken()` function is a more appropriate analogue of `getStETHByWstETH()` at https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L87. It performs calculations using the default `10 ** 18` value. Example:
```solidity
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
    return IWStETH(WST_ETH).stEthPerToken();
}
```

These changes will make the code more readable and easier to understand.

## [05] Function Input Parameter Check

The `_safEthAmount` parameter of the `unstake()` function at https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108 does not have a zero check. Add check for the `_safEthAmount` parameter. For example:
```solidity
require(_safEthAmount != 0, "Unstaking zero amount");
```

The `_weight` parameter of the `adjustWeight()` function at https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L184 does not have any checks. Adjusting a `_weight` with a very large amount may cause arithmetic overflow in the system logic. Add check for the `_weight` parameter. For example:
```solidity
require(_weight < 1e20, "Weight is to large");
```

The `_derivativeIndex` parameter of the `adjustWeight()` function at https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L166 and `setMaxSlippage()` function at https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L203 do not have out of range checks. To prevent potential issues, add checks for the `_derivativeIndex` parameter. For example:
```solidity
require(_derivativeIndex < derivativeCount, "Index is out of range");
```

The `_minAmount` parameter of the `setMinAmount()` function at https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214 should be greater than zero and less than `maxAmount`. Add check for the `_minAmount` parameter. For example:
```solidity
require(_minAmount != 0, "Should not be zero");
require(_minAmount < maxAmount, "Shoul be less than maxAmount");
```
The `_maxAmount` parameter of the `setMaxAmount()` function at https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223 should be greater than `_minAmount`. Add check for the `_maxAmount` parameter. For example:
```solidity
require(_maxAmount > minAmount, "Should be greater than minAmount");
```

Consider adding these checks to ensure that input parameters are within the expected range.

## [06] Unused Variable

The `_amount` variables at https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86 and https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L111 are not used and should be removed. Solidity supports input parameters without names. Example:
```solidity
function ethPerDerivative(uint256) public view returns (uint256) {
    return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
}
```

## [07] Incorrect Comments

The comment for the `adjustWeight()` function at https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L158 is incorrect and mistakenly copied from the function `addDerivative()`. Update the comment to accurately describe the `adjustWeight()` function.

Based on `_weight` comment at https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L163 is not clear that a direvative can be paused by setting `_weight` to zero. Update the comment to clarify this point.

## [08] Use Batch Updates for Weight Adjustments

The `adjustWeight()` function at https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165 currently updates only one weight at a time. If you have a large number of derivatives and need to update multiple weights in a single transaction, consider implementing a batch update function that takes an array of indices and weights.