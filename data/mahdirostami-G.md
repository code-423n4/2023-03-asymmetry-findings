## Emitting Storage Variables

You can save an SLOAD (~100 gas) by emiting local variables over
storage variables when they have the same value.
Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead.

4 instances
affected code:

```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L216
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L225
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L234
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L243
```

MITIGATION:

```
-    emit ChangeMinAmount(minAmount);
+    emit ChangeMinAmount(_minAmount); 
```

## State variables should be cached in stack variables rather than re-reading them from storage

14 instances
affected code:

```
cache totalWeight
uint256 ethAmount = (msg.value * weight) / totalWeight;
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L88
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L149-L150
```

```
cache msg.value
require(msg.value >= minAmount, "amount too low");
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L65
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L66
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L88
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L100
```

```
cache derivativeCount
for (uint i = 0; i < derivativeCount; i++)
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L84
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L147
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L171
```

```
cache derivatives[i].balance()
(derivatives[i].ethPerDerivative(derivatives[i].balance()) * derivatives[i].balance()) / 10 ** 18;
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L73-L75
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L141-L142
```

MITIGATION:

```
-    for (uint i = 0; i < derivativeCount; i++) {
-       uint256 weight = weights[i];
-       IDerivative derivative = derivatives[i];
-       if (weight == 0) continue;
-       uint256 ethAmount = (msg.value * weight) / totalWeight;
+    uint256 _totalWeight = totalWeight;
+    uint256 _derivativeCount = derivativeCount;
+    uint256 msgvalue = msg.value;
+    for (uint i = 0; i < _derivativeCount ; i++) {
+       uint256 weight = weights[i];
+       IDerivative derivative = derivatives[i];
+       if (weight == 0) continue; 
+       uint256 ethAmount = (msgvalue  * weight) / _totalWeight;
```

## Avoid extra computation

1. First checking totalSupply prevent doing extra for loop.

affected code: 

```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71-L81

-        uint256 underlyingValue = 0;
-        for (uint i = 0; i < derivativeCount; i++)
-        underlyingValue +=
-        (derivatives[i].ethPerDerivative(derivatives[i].balance()) * derivatives[i].balance()) / 10 ** 18;
-        uint256 totalSupply = totalSupply();
-        uint256 preDepositPrice; 
-        if (totalSupply == 0)
-            preDepositPrice = 10 ** 18; 
-        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

+    uint256 totalSupply = totalSupply();
+    uint256 preDepositPrice;
+    if (totalSupply == 0)
+      preDepositPrice = 10 ** 18; 
+    else {
+      uint256 underlyingValue = 0; 
+      for ( uint i = 0; i < derivativeCount; i++ )
+        underlyingValue +=
+          (derivatives[i].ethPerDerivative(derivatives[i].balance()) * // @audit gas catch derivatives[i].balance()
+            derivatives[i].balance()) /
+          10 ** 18;
+      preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
+     }
```
2. First checking weight prevent creating extra IDerivative.

affected code: 

```
-        for (uint i = 0; i < derivativeCount; i++) {
-            uint256 weight = weights[i];
-            IDerivative derivative = derivatives[i];
-            if (weight == 0) continue;
-            uint256 ethAmount = (msg.value * weight) / totalWeight; 
+        for (uint i = 0; i < derivativeCount; i++) {
+      	     uint256 weight = weights[i];
+            if (weight == 0) continue;
+            IDerivative derivative = derivatives[i];
+            uint256 ethAmount = (msg.value * weight) / totalWeight;  check 
```
3. First checking ethAmountToRebalance prevent creating extra for loop.

affected code:

```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L147-L149

-        uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
-
-        for (uint i = 0; i < derivativeCount; i++) {
-            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;

+    	 if (ethAmountToRebalance != 0) {
+            for (uint i = 0; i < derivativeCount; i++) {
+            if (weights[i] == 0) continue;
```
3. delete for loop

affected code:

```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L190-L193

-        uint256 localTotalWeight = 0;
-        for (uint256 i = 0; i < derivativeCount; i++)
-            localTotalWeight += weights[i];
-        totalWeight = localTotalWeight;
+        totalWeight = totalWeight + _weight;

```

## Using unchecked blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block see resource


2 instances
affected code:

```
uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L122

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L145
```

MITIGATION:

```
x = b - a => unchecked { x = b - a }
```
## Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a
 normal user tries to pay the function. Marking the function as payable will 
lower the gas cost for legitimate callers because the compiler will not include 
checks for whether a payment was provided. The extra opcodes avoided are
 CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),
DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of 
about 21 gas per call to the function, in addition to the extra deployment cos

12 instances
affected code:
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L138
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L165
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L214
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L223
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L232
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L241
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L60
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56
```

## Short-circuiting

Short-circuiting is a strategy we can make use of when an operation makes use of either || or &&. This pattern works by ordering the lower-cost operation first so that the higher-cost operation may be skipped (short-circuited) if the first operation evaluates to true.
 
1 instances
affected code:
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L147-L149

            rocketDepositPool.getBalance() + _amount <=
            rocketDAOProtocolSettingsDeposit.getMaximumDepositPoolSize() &&
            _amount >= rocketDAOProtocolSettingsDeposit.getMinimumDeposit();

```
MITIGATION:
Use lower-cost operation first