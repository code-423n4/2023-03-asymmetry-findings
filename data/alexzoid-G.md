## [G-01] Storage Variable Accessed Multiple Times

The `derivativeCount` storage variable is accessed multiple times within for-loops, which can lead to inefficient gas usage:

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71
```solidity
for (uint i = 0; i < derivativeCount; i++)
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84
```solidity
for (uint i = 0; i < derivativeCount; i++) {
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113
```solidity
for (uint256 i = 0; i < derivativeCount; i++) {
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140
```solidity
for (uint i = 0; i < derivativeCount; i++) {
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147
```solidity
for (uint i = 0; i < derivativeCount; i++) {
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171
```solidity
for (uint256 i = 0; i < derivativeCount; i++)
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191
```solidity
for (uint256 i = 0; i < derivativeCount; i++)
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L186-L188 
```solidity
derivatives[derivativeCount] = IDerivative(_contractAddress);
weights[derivativeCount] = _weight;
derivativeCount++;
```

Additionally, the `derivatives[i]` and `weights[i]` storage variables are accessed multiple times:

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L73-L74
```solidity
// Getting underlying value in terms of ETH for each derivative
for (uint i = 0; i < derivativeCount; i++)
    underlyingValue +=
        (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
            derivatives[i].balance()) /
        10 ** 18;
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L115-L118
```solidity
for (uint256 i = 0; i < derivativeCount; i++) {
    // withdraw a percentage of each asset based on the amount of safETH
    uint256 derivativeAmount = (derivatives[i].balance() *
        _safEthAmount) / safEthTotalSupply;
    if (derivativeAmount == 0) continue; // if derivative empty ignore
    derivatives[i].withdraw(derivativeAmount);
}
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L141-L142
```solidity
for (uint i = 0; i < derivativeCount; i++) {
    if (derivatives[i].balance() > 0)
        derivatives[i].withdraw(derivatives[i].balance());
}
```
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L148-L149
```solidity
for (uint i = 0; i < derivativeCount; i++) {
    if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
    uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
        totalWeight;
    // Price will change due to slippage
    derivatives[i].deposit{value: ethAmount}();
}
```

Consider caching these variables in memory to optimize gas usage.

## [G-02] Check `ethAmountToRebalance` Outside a Loop

The `ethAmountToRebalance` variable at https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L148 could be checked once before the for-loop. Example:
```solidity
uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
require(ethAmountToRebalance != 0, "Nothing to rebalance");
```

## [G-03] Optimize `totalWeight` Calculation 

The `totalWeight` calculation in the `adjustWeight()` function at https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L170-L173 could be optimized. You can simply subtract the old weight and add the new weight, like this:
```solidity
totalWeight -= weights[_derivativeIndex] + _weight;
weights[_derivativeIndex] = _weight; 
```

The `totalWeight` calculation in the `addDerivative()` function at lines https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L190-L193 could be optimized as well. You can simply add the new weight, like this:
```solidity
totalWeight += _weight;
```