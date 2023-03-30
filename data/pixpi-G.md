## Gas Optimizations

---

|                 | Issue                                                                              | Instances |
| --------------- | :--------------------------------------------------------------------------------- | :-------: |
| [GAS-1](#GAS-1) | Refactor `weights` to be normalized                                                |     1     |
| [GAS-2](#GAS-2) | Cache `derivativeCount` in memory before using inside `for` loop                   |     7     |
| [GAS-3](#GAS-3) | Cache calls to `derivatives[i].balance()` in memory before using inside `for` loop |     1     |
| [GAS-4](#GAS-4) | Cache `totalWeight` in memory before using inside `for` loop                       |     2     |
| [GAS-5](#GAS-5) | Refactor conditional checks inside `for` loop                                      |     1     |
| [GAS-6](#GAS-6) | Use `unchecked` for `derivativeCount` increment                                    |     1     |

In the following report gas optimizations are sorted by the estimated amount of gas savings.

All gas estimations were calculated by using Remix IDE and deploying two separate contracts that differ only in one small part which was being tested.

Settings used: `{version: "0.8.13", settings: {optimizer: {enabled: true, runs: 100000}`

---

## <a name="GAS-1"></a>[GAS-1] Refactor `weights` to be normalized

Current implementation uses `totalWeight` to keep track of the total sum of all weights.

However, `totalWeight` is updated only in `addDerivative()` & `adjustWeight()` functions, but used inside the `stake()` function's `for` loop to calculate normalized weights.

It's safe to assume that function `stake()` will be used much more frequently than `addDerivative()` & `adjustWeight()`.

Therefore, the code could be refactored in a way that saves gas by calculating and saving normalized values for `weights`
in `addDerivative()` & `adjustWeight()` functions.

This allows to remove `totalWeight` state variable which will save gas both during deployment and code execution.

Estimated gas savings for one `stake()` function call: 2163 gas + 163 gas \* (number of non-zero weights - 1)

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L187-L193

Recommendation code:

```solidity
File: contracts/SafEth/SafEth.sol
+       unit256 private constant WEIGHT_BASE = 10000; // choose presicion

- 88:            uint256 ethAmount = (msg.value * weight) / totalWeight;
+ 88:            uint256 ethAmount = (msg.value * weight) / WEIGHT_BASE;

- 149:           uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
- 150:               totalWeight;
+ 149:           uint256 ethAmount = (ethAmountToRebalance * weights[i]) / WEIGHT_BASE;

// refactored addDerivative()
- 187:        weights[derivativeCount] = _weight;
- 188:        derivativeCount++;
- 189:
- 190:        uint256 localTotalWeight = 0;
- 191:        for (uint256 i = 0; i < derivativeCount; i++)
- 192:            localTotalWeight += weights[i];
- 193:        totalWeight = localTotalWeight;

+ 187:        addWeight(_weight);
+             unchecked {
+                 derivativeCount++;
+             }

// refactored adjustWeight()
    function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) public onlyOwner {
        require(_weight <= WEIGHT_BASE, "Wrong weight");
        // limit weight to be at least 1%
        require(_weight >= WEIGHT_BASE / 100 || _weight == 0, "Wrong weight");
        require(derivativeCount > 0, "No derivatives created");
        require(_derivativeIndex < derivativeCount, "Wrong derivative index");

        if (!_checkNonZeroSum()) {
            require(_weight == WEIGHT_BASE, "Wrong weight, zero sum");
        }

        uint256 weightsLen = derivativeCount;
        uint256 oldWeight = weights[_derivativeIndex];
        uint256 remainder = WEIGHT_BASE - _weight;

        for (uint256 i = 0; i < weightsLen; ) {
            if (i != _derivativeIndex) {
                unchecked {
                    weights[i] = weights[i] * (WEIGHT_BASE - _weight) / (WEIGHT_BASE - oldWeight);
                    remainder -= weights[i];
                    i++;
                }
            } else {
               weights[_derivativeIndex] = _weight;
               unchecked {
                    i++;
                }
            }
        }

        // assigning remainder to any non-zero weight
        // in order to keep WEIGHT_BASE invariant
        for (uint256 i = 0; i < weightsLen;) {
            if (weights[i] != 0) {
                weights[i] += remainder;
                break;
            } else {
                unchecked {
                    i++;
                }
            }
        }
        emit WeightChange(_derivativeIndex, _weight);
    }

// helper functions
    // check if all current weights are not set to 0
    function _checkNonZeroSum() internal view returns (bool nonZeroSum) {
        uint256 weightsLen = derivativeCount;

        for (uint256 i = 0; i < weightsLen;) {
            if (weights[i] > 0 ) {
                nonZeroSum = true;
                break;
            }
            unchecked {
                i++;
            }
        }
    return nonZeroSum;
    }

    function addWeight(uint256 newWeight) internal {
        require(newWeight <= WEIGHT_BASE, "Wrong weight");
        // limit weight to be at least 1%
        require(newWeight >= WEIGHT_BASE / 100, "Wrong weight");

        // first weight can only be equal to WEIGHT_BASE
        if (derivativeCount == 0) {
            require(newWeight == WEIGHT_BASE, "Wrong first weight");
            weights[0] = newWeight;
        } else {
            if (!_checkNonZeroSum()) {
                require(newWeight == WEIGHT_BASE, "Wrong weight, zero sum");
                weights[derivativeCount] = newWeight;
            } else {
                uint256 weightsLen = derivativeCount;
                uint256 remainder = WEIGHT_BASE - newWeight;

                for (uint256 i = 0; i < weightsLen;) {
                    unchecked {
                        weights[i] = weights[i] * (WEIGHT_BASE - newWeight) / WEIGHT_BASE;
                        remainder -= weights[i];
                        i++;
                    }
                }

                weights[derivativeCount] = newWeight + remainder;
            }
        }
    }
```

---

## <a name="GAS-2"></a>[GAS-2] Cache `derivativeCount` in memory before using inside `for` loop

Every conditional check `i < derivativeCount` inside `for` loop costs 100 gas due to warm access to the state variable.

Therefore, it is recommended to cache `derivativeCount` in memory before using in the loop.

Total gas savings: 100 gas \* `derivativeCount` \* 7

7 instances in 1 file:

```solidity
File: contracts/SafEth/SafEth.sol:
  71:        for (uint i = 0; i < derivativeCount; i++)

  84:        for (uint i = 0; i < derivativeCount; i++) {

  113:        for (uint256 i = 0; i < derivativeCount; i++) {

  140:        for (uint i = 0; i < derivativeCount; i++) {

  147:        for (uint i = 0; i < derivativeCount; i++) {

  171:        for (uint256 i = 0; i < derivativeCount; i++)

  191:        for (uint256 i = 0; i < derivativeCount; i++)
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71

Recommendation code example:

```solidity
File: contracts/SafEth/SafEth.sol:
- 71:        for (uint i = 0; i < derivativeCount; i++)

+ 71:        uint256 memoryCount = derivativeCount;
+            for (uint i = 0; i < memoryCount; i++)

```

---

## <a name="GAS-3"></a>[GAS-3] Cache calls to `derivatives[i].balance()` in memory before using inside `for` loop

There are two `derivatives[i].balance()` calls in every loop iteration. Caching them in memory will save gas.

Gas savings estimate: 480 gas \* (number of derivatives with positive balance)

1 instance in 1 file:

```solidity
File: contracts/SafEth/SafEth.sol:
  141:            if (derivatives[i].balance() > 0)
  142:                derivatives[i].withdraw(derivatives[i].balance());
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L141-L142

Recommendation code:

```solidity
File: contracts/SafEth/SafEth.sol:
- 141:            if (derivatives[i].balance() > 0)
- 142:                derivatives[i].withdraw(derivatives[i].balance());

+                 uint256 derivativeBalance = derivatives[i].balance();
+ 141:            if (derivativeBalance > 0)
+ 142:                derivatives[i].withdraw(derivativeBalance);
```

---

## <a name="GAS-4"></a>[GAS-4] Cache `totalWeight` in memory before using inside `for` loop

Every access to `totalWeight` variable inside `for` loop costs 100 gas (warm access to the state variable).

Therefore, it is recommended to cache it in memory before calling.

Total gas savings: 100 gas \* `derivativeCount` \* 2

2 instances in 1 file:

```solidity
File: contracts/SafEth/SafEth.sol:
  88:            uint256 ethAmount = (msg.value * weight) / totalWeight;

  150:                totalWeight;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L88

Recommendation code example:

```solidity
File: contracts/SafEth/SafEth.sol:
+            uint256 totalWeightCache = totalWeight;
  84:        for (uint i = 0; i < derivativeCount; i++) {

- 88:            uint256 ethAmount = (msg.value * weight) / totalWeight;

+ 88:            uint256 ethAmount = (msg.value * weight) / totalWeightCache;
```

---

## <a name="GAS-5"></a>[GAS-5] Refactor conditional checks inside `for` loop

Due to the fact that `ethAmountToRebalance` doesn't change inside `for` loop, conditional check `ethAmountToRebalance == 0` in every iteration just wastes gas.

Therefore, it is suggested to refactor the code and move that conditional check outside `for` loop.

Gas savings estimate:

- if `ethAmountToRebalance != 0` : 25 gas \* `derivativeCount`
- if `ethAmountToRebalance == 0` : 2400 gas \* `derivativeCount`

1 instance in 1 file:

```solidity
File: contracts/SafEth/SafEth.sol:
  147:        for (uint i = 0; i < derivativeCount; i++) {
  148:            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L147-L148

Recommendation code:

```solidity
File: contracts/SafEth/SafEth.sol:
+            if (ethAmountToRebalance != 0)
  147:           for (uint i = 0; i < derivativeCount; i++) {
- 148:               if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
+ 148:               if (weights[i] == 0 ) continue;
```

---

## <a name="GAS-6"></a>[GAS-6] Use `unchecked` for `derivativeCount` increment

It is safe to assume that `derivativeCount` will never reach the maximum of `uint256` type.

Thus, it is possible to save gas by disabling overflow/underflow checks.

Gas savings: 83 gas

1 instance in 1 file:

```solidity
File: contracts/SafEth/SafEth.sol:
  188:        derivativeCount++;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L188

Recommendation code:

```solidity
File: contracts/SafEth/SafEth.sol:
- 188:        derivativeCount++;

+ 188:        unchecked {
+                 derivativeCount++;
+              }
```

---
