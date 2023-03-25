# The calculation of derivativeAmount in unstake function which could potentially lead to an integer overflow, which can cause unexpected behavior and loss of funds.

##  Affected Code
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108-L129

```
uint256 derivativeAmount = (derivatives[i].balance() * _safEthAmount) / safEthTotalSupply;
```

## Description and Impact
The multiplication of `derivatives[i].balance()` and _safEthAmount may result in a number that exceeds the maximum or minimum value.

For example, if the user wants to `unstake` 10 token, at this time, the `_safEthAmount` should be 10. 
If the total supply of SAFETH is 1,000,000, then the line of code 
`uint256 derivativeAmount = (derivatives[i].balance() * _safEthAmount) / safEthTotalSupply;` would result in derivativeAmount being calculated as `derivatives[i].balance() * 10 / 1,000,000`.

If `derivatives[i].balance()` is very large, such as 2^256 - 1 (the maximum value of a uint256), then the multiplication may result in an integer overflow, and then the `derivativeAmount` to become 0.

### Fix
Use SafeMath Library
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeMath.sol