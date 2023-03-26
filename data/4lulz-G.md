## Redundant calculations `SafEth`

Functions [`adjustWeight`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L165) and ['addDerivative'](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182) recalculates `totalWeight` iterating over all `weights`.
The more efficient way:
  ### [`adjustWeight`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L165):
`totalWeight = totalWeight - weights[_derivativeIndex] + _weight` 
  ### [`addDerivative`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182):
`totalWeight += _weight`

## Redundant calculations `Reth`

The [`ethPerDerivative`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L215) function's return expression makes a redundant multiplication and then division.

`(poolPrice() * 10 ** 18) / (10 ** 18)` will always have the same value as `poolPrice()`. 
## Storage to memory

Contract gas consumption can be reduced by saving storage variables in memory (reducing `SLOAD`s).
There is a couple of such instances:

 ### `totalWeight`

Moving `totalWeight` to memory before loops would save gas

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L88

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L147

### `derivativeCount`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L186

### Derivative mapping

Save into memory address:
```Solidity
IDerivative derivative = derivatives[i];
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113

Function [stake()](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L63) saves a derivative address in [the second loop](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L86), although there are 3 `SLOAD` operations for each address in the first loop.

Recommended to store whole `derivatives` mapping into the memory before all further usage. E.g. in the first loop:
```Solidity

IDerivative[] memory memoryDerivatives = new IDerivative[](derivativeLength); 

for (uint i = 0; i < derivativeCount; i++){
    memoryDerivatives[i] = derivatives[i];
    underlyingValue +=
        (memoryDerivatives[i].ethPerDerivative(memoryDerivatives[i].balance()) *
            memoryDerivatives[i].balance()) /
        10 ** 18;
}
```

The same could be done in the [`rebalanceToWeights`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L138) function.

This approach could be improved (easier copy to memory, cleaner [`addDerivative function`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182)) if you change the [`derivatives`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEthStorage.sol#L22) mapping to an array; since mapping is already being used like an array.
This would also let you remove the [`derivativeCount`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEthStorage.sol#L18) variable.
