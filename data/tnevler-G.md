# Report
## Gas Optimizations ##
### [G-1]: Cache state variable in memory and not re-read it from the storage
**Context:**

1. ```for (uint i = 0; i < derivativeCount; i++)``` [L71](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71) (derivativeCount)
1. ```for (uint i = 0; i < derivativeCount; i++) {``` [L84](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84) (derivativeCount)
1. ```for (uint256 i = 0; i < derivativeCount; i++) {``` [L113](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113) (derivativeCount)
1. ```for (uint i = 0; i < derivativeCount; i++) {``` [L140](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140) (derivativeCount)
1. ```for (uint i = 0; i < derivativeCount; i++) {``` [L147](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147) (derivativeCount)
1. ```for (uint256 i = 0; i < derivativeCount; i++)``` [L171](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171) (derivativeCount)
1. ```derivatives[derivativeCount] = IDerivative(_contractAddress);``` [L185](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L185) (derivativeCount)
1. ```weights[derivativeCount] = _weight;``` [L186](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L186) (derivativeCount)
1. ```for (uint256 i = 0; i < derivativeCount; i++)``` [L190](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L190) (derivativeCount)
1. ```emit DerivativeAdded(_contractAddress, _weight, derivativeCount);``` [L193](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L193) (derivativeCount)

**Description:**

Every reading from storage costs about 100 gas while every reading from memory costs only 3 gas. If state variable referred more than once within a function then it is cheaper to cache it in local memory (100 gas) and then read it from memory wnen it is neaded (3 gas) rather than read state variable from storage everytime (100 gas).

**Recommendation:**

Example how to fix. Change:
```
for (uint i = 0; i < derivativeCount; i++)
```

to:
```
    uint256 _derivativeCount = derivativeCount;
    for (uint i = 0; i < _derivativeCount; i++)
```

### [G-2]: Unnecessary loop
**Context:**

1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191-L193
1. https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171-L172

**Description:**

Total weight of all derivatives is already stored in totalWeight. It is unnecessary to calculate it in loop again. 

**Recommendation:**

Instead of [this](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L190-L193):
```
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
```
Do this:
```
        totalWeight += _weight;
```

Instead of [this](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L169-L173):
```
        weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
```
Do this:
```
        uint256 oldWeight = weights[_derivativeIndex];
        weights[_derivativeIndex] = _weight;
        totalWeight -= oldWeight;
        totalWeight += _weight;
```

### [G-3]: Place subtractions where the operands can't underflow in unchecked {} block
**Context:**

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L201 (checked in L200)

**Description:**

Some gas can be saved by using an unchecked {} block if an underflow isn't possible because of a previous require() or if-statement.

### [G-4]: Cache a value from a mapping/array in local memory
**Context:**

1. ```(derivatives[i].ethPerDerivative(derivatives[i].balance()) *``` [L73](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L73) (derivatives[i])
1. ``` derivatives[i].balance()) /``` [L74](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L74) (derivatives[i])
1. ```if (derivatives[i].balance() > 0)``` [L141](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L141) (derivatives[i])
1. ```derivatives[i].withdraw(derivatives[i].balance());``` [L142](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L142) (derivatives[i])
1. ```if (weights[i] == 0 || ethAmountToRebalance == 0) continue;``` [L148](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L148) (weights[i])
1. ```uint256 ethAmount = (ethAmountToRebalance * weights[i]) /``` [L149](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L149) (weights[i])

**Description:**

If you read value from mapping/array more than once within a function then it is cheaper to cache it in local memory and then read it from memory wnen it is neaded. This will save about 100 gas.

**Recommendation:**
Example how to fix. Change:
```
        for (uint i = 0; i < derivativeCount; i++)
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                    derivatives[i].balance()) /
                10 ** 18;

```

to:
```
        for (uint i = 0; i < derivativeCount; i++)
            IDerivative derivative = derivatives[i];
            underlyingValue +=
                (derivative.ethPerDerivative(derivative.balance()) *
                    derivative.balance()) /
                10 ** 18;

```