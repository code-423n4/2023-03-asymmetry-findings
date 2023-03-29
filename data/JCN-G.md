# Summary
All optimizations (*with the exception of issue [G-10](#adding-a-derivative-will-increase-gas-costs-for-users-staking-and-unstaking-even-after-the-derivative-is-removed)*) were benchmarked via the protocol's tests, i.e. using the following config: `solc version 0.8.13`, `optimizer on`, and `100_000 runs`.

Below are the overall average gas savings for the tested functions (with all the optimizations applied):
| Function |    Before   |    After   | Avg Gas Savings |
| ------ | -------- | -------- | ------- |
| stake |  527253  |  511519  |  15734 | 
| unstake |  516303  |  513131  |  3172 |  
| addDerivative |  102462  |  99120  |  3342 | 
| adjustWeight|  46519  |  39195  |  7324 | 
| rebalanceToWeights|  727618  |  717607  |  10011 | 
| setMaxAmount|  37155  |  37146  | 9 | 
| setMaxSlippage|  58574  |  58532  | 42 | 
| setMinAmount|  37099  |  37090  |  9 | 
| setPauseStaking|  54296  |  54263  | 33 | 
| setPauseUnstaking|  37243  |  37193  |  50 | 
| deposit|  176462  |  176124  | 338 | 
| withdraw|  181057  |  180930  | 127 | 
| adminWithdrawDerivative|  195722  |  195594  | 128 |

**Total gas saved across all listed functions: 40319**

*Notes*: 

- The [`gasReporter`](#gasreporter-output-with-all-optimizations-applied) output and [diffs](#diff-for-safethsol-with-all-optimizations-applied) with **all the optimizations applied** (*with the exception of issue [G-10](#adding-a-derivative-will-increase-gas-costs-for-users-staking-and-unstaking-even-after-the-derivative-is-removed)*) can be found at the end of the report.
- Some code snippets may be truncated to save space. Code snippets may also be accompanied by `@audit` tags in comments to aid in explaining the issue.

## Gas Optimizations
| Number |Issue|Instances|
|-|:-|:-:|
| [G-01](#state-variables-can-be-cached-instead-of-re-reading-them-from-storage) | State variables can be cached instead of re-reading them from storage | 22 |
| [G-02](#avoid-emitting-storage-values) | Avoid emitting storage values | 4 |
| [G-03](#refactor-code-to-avoid-unecessary-for-loop) | Refactor code to avoid unecessary for loop | 2 |
| [G-04](#return-values-from-external-calls-can-be-cached-to-avoid-unecessary-call) | Return values from external calls can be cached to avoid unecessary CALL | 2 |
| [G-05](#declare-keccak256-hashes-as-constants-to-avoid-computing-hash-each-time-the-specified-functions-are-called) | Declare keccak256 hashes as constants to avoid computing hash each time the specified functions are called | 6 |
| [G-06](#expressions-can-be-unchecked-if-gauranteed-to-not-underflow-due-to-previous-require-statement) | Expressions can be unchecked if gauranteed to not underflow due to previous require statement | 1 |
| [G-07](#use-assembly-for-value-transfer-and-success-check) | Use assembly for value transfer and success check | 5 | 
| [G-08](#use-assembly-to-emit-events) | Use assembly to emit events | 10 | 
| [G-09](#use-assembly-for-loops) | Use assembly for loops | 3 | 
| [G-10](#adding-a-derivative-will-increase-gas-costs-for-users-staking-and-unstaking-even-after-the-derivative-is-removed) | Adding a derivative will increase gas costs for users staking and unstaking even after the derivative is removed | - | 

## State variables can be cached instead of re-reading them from storage
Caching of a state variable replaces each `Gwarmaccess (100 gas)` with a much cheaper stack read.

Total Instances: `22`

***Gas savings will actually be greater since storage slot access is occuring within loops in some instances**

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63-L88

### Cache `derivativeCount`, `derivatives[i]`, and `totalWeight` to save ~ 6 SLOADs

*More SLOADs would actually be saved since storage slot access is occuring within a loop.*
```solidity
File: contracts/SafEth/SafEth.sol
63:    function stake() external payable {
64:        require(pauseStaking == false, "staking is paused");
65:        require(msg.value >= minAmount, "amount too low");
66:        require(msg.value <= maxAmount, "amount too high");
67:
68:        uint256 underlyingValue = 0;
69:
70:        // Getting underlying value in terms of ETH for each derivative
71:        for (uint i = 0; i < derivativeCount; i++) // @audit: sload on every iteration
72:            underlyingValue +=
73:                (derivatives[i].ethPerDerivative(derivatives[i].balance()) * // @audit: 1st and 2nd sload
74:                    derivatives[i].balance()) / // @audit: 3rd sload
75:                10 ** 18;
76:
77:        uint256 totalSupply = totalSupply();
78:        uint256 preDepositPrice; // Price of safETH in regards to ETH
79:        if (totalSupply == 0)
80:            preDepositPrice = 10 ** 18; // initializes with a price of 1
81:        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
82:
83:        uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
84:        for (uint i = 0; i < derivativeCount; i++) { // @audit: sload on every iteration
85:            uint256 weight = weights[i];
86:            IDerivative derivative = derivatives[i]; // @audit: unecessary sload if weight == 0, move below if statement
87:            if (weight == 0) continue;
88:            uint256 ethAmount = (msg.value * weight) / totalWeight; // @audit: sload on every iteration
89:
90:            // This is slightly less than ethAmount because slippage
91:            uint256 depositAmount = derivative.deposit{value: ethAmount}();
92:            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
93:                depositAmount
94:            ) * depositAmount) / 10 ** 18;
95:            totalStakeValueEth += derivativeReceivedEthValue;
96:        }
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..51a495b 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -68,11 +68,14 @@ contract SafEth is
         uint256 underlyingValue = 0;

         // Getting underlying value in terms of ETH for each derivative
-        for (uint i = 0; i < derivativeCount; i++)
+        uint256 _derivativeCount = derivativeCount;
+        for (uint i = 0; i < _derivativeCount; i++) {
+            IDerivative derivative = derivatives[i];
             underlyingValue +=
-                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
-                    derivatives[i].balance()) /
+                (derivative.ethPerDerivative(derivative.balance()) *
+                    derivative.balance()) /
                 10 ** 18;
+        }

         uint256 totalSupply = totalSupply();
         uint256 preDepositPrice; // Price of safETH in regards to ETH
@@ -81,13 +84,14 @@ contract SafEth is
         else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

         uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
-        for (uint i = 0; i < derivativeCount; i++) {
+        uint256 _totalWeight;
+        for (uint i = 0; i < _derivativeCount; i++) {
             uint256 weight = weights[i];
-            IDerivative derivative = derivatives[i];
             if (weight == 0) continue;
-            uint256 ethAmount = (msg.value * weight) / totalWeight;
+            uint256 ethAmount = (msg.value * weight) / _totalWeight;

             // This is slightly less than ethAmount because slippage
+            IDerivative derivative = derivatives[i];
             uint256 depositAmount = derivative.deposit{value: ethAmount}();
             uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
                 depositAmount
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108-L119

### Cache `derivativeCount` and `derivatives[i]` to save ~ 3 SLOADs

*More SLOADs would actually be saved since storage slot access is occuring within a loop.*
```solidity
File: contracts/SafEth/SafEth.sol
108:    function unstake(uint256 _safEthAmount) external {
109:        require(pauseUnstaking == false, "unstaking is paused");
110:        uint256 safEthTotalSupply = totalSupply();
111:        uint256 ethAmountBefore = address(this).balance;
112:
113:        for (uint256 i = 0; i < derivativeCount; i++) { // @audit: sload on every iteration
114:            // withdraw a percentage of each asset based on the amount of safETH
115:            uint256 derivativeAmount = (derivatives[i].balance() * // @audit: 1st sload
116:                _safEthAmount) / safEthTotalSupply;
117:            if (derivativeAmount == 0) continue; // if derivative empty ignore
118:            derivatives[i].withdraw(derivativeAmount); // @audit: 2nd sload
119:        }
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..69f0a72 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -109,13 +109,15 @@ contract SafEth is
         require(pauseUnstaking == false, "unstaking is paused");
         uint256 safEthTotalSupply = totalSupply();
         uint256 ethAmountBefore = address(this).balance;
+        uint256 _derivativeCount = derivativeCount;

-        for (uint256 i = 0; i < derivativeCount; i++) {
+        for (uint256 i = 0; i < _derivativeCount; i++) {
             // withdraw a percentage of each asset based on the amount of safETH
-            uint256 derivativeAmount = (derivatives[i].balance() *
+            IDerivative derivative = derivatives[i];
+            uint256 derivativeAmount = (derivative.balance() *
                 _safEthAmount) / safEthTotalSupply;
             if (derivativeAmount == 0) continue; // if derivative empty ignore
-            derivatives[i].withdraw(derivativeAmount);
+            derivative.withdraw(derivativeAmount);
         }
         _burn(msg.sender, _safEthAmount);
         uint256 ethAmountAfter = address(this).balance;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138-L155

### Cache `derivativeCount`, `derivatives[i]`, `weights[i]`, and `totalWeight` to save ~ 8 SLOADs

*More SLOADs would actually be saved since storage slot access is occuring within a loop.*
```solidity
File: contracts/SafEth/SafEth.sol
138:    function rebalanceToWeights() external onlyOwner {
139:        uint256 ethAmountBefore = address(this).balance;
140:        for (uint i = 0; i < derivativeCount; i++) { // @audit: sload on every iteration
141:            if (derivatives[i].balance() > 0) // @audit: 1st sload
142:                derivatives[i].withdraw(derivatives[i].balance()); // @audit: 2nd and 3rd sload
143:        }
144:        uint256 ethAmountAfter = address(this).balance;
145:        uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
146:
147:        for (uint i = 0; i < derivativeCount; i++) { // @audit: sload on every iteration
148:            if (weights[i] == 0 || ethAmountToRebalance == 0) continue; // @audit: 1st sload
149:            uint256 ethAmount = (ethAmountToRebalance * weights[i]) / // @audit: 2nd sload
150:                totalWeight; // @audit: sload on every iteration
151:            // Price will change due to slippage
152:            derivatives[i].deposit{value: ethAmount}();
153:        }
154:        emit Rebalanced();
155:    }
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..51ebe34 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -137,17 +137,21 @@ contract SafEth is
     */
     function rebalanceToWeights() external onlyOwner {
         uint256 ethAmountBefore = address(this).balance;
-        for (uint i = 0; i < derivativeCount; i++) {
-            if (derivatives[i].balance() > 0)
-                derivatives[i].withdraw(derivatives[i].balance());
+        uint256 _derivativeCount = derivativeCount;
+        for (uint i = 0; i < _derivativeCount; i++) {
+            IDerivative derivative = derivatives[i];
+            if (derivative.balance() > 0)
+                derivative.withdraw(derivative.balance());
         }
         uint256 ethAmountAfter = address(this).balance;
         uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
+        uint256 _totalWeight = totalWeight;

-        for (uint i = 0; i < derivativeCount; i++) {
-            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
-            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
-                totalWeight;
+        for (uint i = 0; i < _derivativeCount; i++) {
+            uint256 weight = weights[i];
+            if (weight == 0 || ethAmountToRebalance == 0) continue;
+            uint256 ethAmount = (ethAmountToRebalance * weight) /
+                _totalWeight;
             // Price will change due to slippage
             derivatives[i].deposit{value: ethAmount}();
         }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175

### Cache `derivativeCount` to save 1 SLOAD

*More SLOADs would actually be saved since storage slot access is occuring within a loop.*
```solidity
File: contracts/SafEth/SafEth.sol
165:    function adjustWeight(
166:        uint256 _derivativeIndex,
167:        uint256 _weight
168:    ) external onlyOwner {
169:        weights[_derivativeIndex] = _weight;
170:        uint256 localTotalWeight = 0;
171:        for (uint256 i = 0; i < derivativeCount; i++) // @audit: sload on every iteration
172:            localTotalWeight += weights[i];
173:        totalWeight = localTotalWeight;
174:        emit WeightChange(_derivativeIndex, _weight);
175:    }
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..b97b4d3 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -167,8 +167,9 @@ contract SafEth is
         uint256 _weight
     ) external onlyOwner {
         weights[_derivativeIndex] = _weight;
+        uint256 _derivativeCount = derivativeCount;
         uint256 localTotalWeight = 0;
-        for (uint256 i = 0; i < derivativeCount; i++)
+        for (uint256 i = 0; i < _derivativeCount; i++)
             localTotalWeight += weights[i];
         totalWeight = localTotalWeight;
         emit WeightChange(_derivativeIndex, _weight);
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-L195

### Cache `derivativeCount` to save 4 SLOADs

*More SLOADs would actually be saved since storage slot access is occuring within a loop.*
```solidity
File: contracts/SafEth/SafEth.sol
182:    function addDerivative(
183:        address _contractAddress,
184:        uint256 _weight
185:    ) external onlyOwner {
186:        derivatives[derivativeCount] = IDerivative(_contractAddress); // @audit: 1st sload
187:        weights[derivativeCount] = _weight; // @audit: 2nd sload
188:        derivativeCount++; // @audit: 3rd sload
189:
190:        uint256 localTotalWeight = 0;
191:        for (uint256 i = 0; i < derivativeCount; i++) // @audit 4th sload + on every iteration
192:            localTotalWeight += weights[i];
193:        totalWeight = localTotalWeight;
194:        emit DerivativeAdded(_contractAddress, _weight, derivativeCount); // @audit 5th sload
195:    }
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..705da8b 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -183,15 +184,17 @@ contract SafEth is
         address _contractAddress,
         uint256 _weight
     ) external onlyOwner {
-        derivatives[derivativeCount] = IDerivative(_contractAddress);
-        weights[derivativeCount] = _weight;
-        derivativeCount++;
+        uint256 _derivativeCount = derivativeCount;
+        derivatives[_derivativeCount] = IDerivative(_contractAddress);
+        weights[_derivativeCount] = _weight;
+        uint256 newDerivativeCount = _derivativeCount + 1;
+        derivativeCount = newDerivativeCount;

         uint256 localTotalWeight = 0;
-        for (uint256 i = 0; i < derivativeCount; i++)
+        for (uint256 i = 0; i < _derivativeCount; i++)
             localTotalWeight += weights[i];
         totalWeight = localTotalWeight;
-        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
+        emit DerivativeAdded(_contractAddress, _weight, newDerivativeCount);
     }
```

## Avoid emitting storage values
In the instances below we can emit calldata values instead of emitting storage values. This will result in using a cheap `CALLDATALOAD` instead of an expensive `SLOAD`.

Total Instances: `4`

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214-L217

### Emit `_minAmount` instead of reading from storage
```solidity
File: contracts/SafEth/SafEth.sol
214:    function setMinAmount(uint256 _minAmount) external onlyOwner {
215:        minAmount = _minAmount;
216:        emit ChangeMinAmount(minAmount); // @audit: unecessary sload, use _minAmount
217:    }
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..4b6ab0a 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -213,7 +213,7 @@ contract SafEth is
     */
     function setMinAmount(uint256 _minAmount) external onlyOwner {
         minAmount = _minAmount;
-        emit ChangeMinAmount(minAmount);
+        emit ChangeMinAmount(_minAmount);
     }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223-L226

### Emit `_maxAmount` instead of reading from storage
```solidity
File: contracts/SafEth/SafEth.sol
223:    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
224:        maxAmount = _maxAmount;
225:        emit ChangeMaxAmount(maxAmount); // @audit: unecessary sload, use _maxAmount
226:    }
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..4b6ab0a 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -222,7 +222,7 @@ contract SafEth is
     */
     function setMaxAmount(uint256 _maxAmount) external onlyOwner {
         maxAmount = _maxAmount;
-        emit ChangeMaxAmount(maxAmount);
+        emit ChangeMaxAmount(_maxAmount);
     }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232-L235

### Emit `_pause` instead of reading from storage
```solidity
File: contracts/SafEth/SafEth.sol
232:    function setPauseStaking(bool _pause) external onlyOwner {
233:        pauseStaking = _pause;
234:        emit StakingPaused(pauseStaking); // @audit: unecessary sload, use _pause
235:    }
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..4b6ab0a 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -231,7 +231,7 @@ contract SafEth is
     */
     function setPauseStaking(bool _pause) external onlyOwner {
         pauseStaking = _pause;
-        emit StakingPaused(pauseStaking);
+        emit StakingPaused(_pause);
     }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241-L244

### Emit `_pause` instead of reading from storage
```solidity
File: contracts/SafEth/SafEth.sol
241:    function setPauseUnstaking(bool _pause) external onlyOwner {
242:        pauseUnstaking = _pause;
243:        emit UnstakingPaused(pauseUnstaking); // @audit: unecessary sload, use _pause
244:    }
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..4b6ab0a 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -240,8 +240,9 @@ contract SafEth is
     */
     function setPauseUnstaking(bool _pause) external onlyOwner {
         pauseUnstaking = _pause;
-        emit UnstakingPaused(pauseUnstaking);
+        emit UnstakingPaused(_pause);
     }
```

## Refactor code to avoid unecessary for loop
Reading storge in a loop is a very gas expensive operation. Given that derivatives can be added to the protocol, `derivativeCount` can increase and result in more storage reads (via more loop iterations). `totalWeight` can be updated using simple expressions instead of using a for loop. Consider refactoring the functions below to avoid using for loops and do less storage reads.

Total Instances: `2`

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175

```solidity
File: contracts/SafEth/SafEth.sol
165:    function adjustWeight(
166:        uint256 _derivativeIndex,
167:        uint256 _weight
168:    ) external onlyOwner {
169:        weights[_derivativeIndex] = _weight;
170:        uint256 localTotalWeight = 0;
171:        for (uint256 i = 0; i < derivativeCount; i++) // @audit: sload on every iteration
172:            localTotalWeight += weights[i]; // @audit: sload on every iteration
173:        totalWeight = localTotalWeight; // @audit: localWeight is totalWeight - (old weight of weights[_derivativeIndex]) + _weight, loop is unecessary
174:        emit WeightChange(_derivativeIndex, _weight);
175:    }
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..817e53a 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -166,11 +166,9 @@ contract SafEth is
         uint256 _derivativeIndex,
         uint256 _weight
     ) external onlyOwner {
+        uint256 oldWeight = weights[_derivativeIndex];
+        totalWeight = totalWeight - oldWeight + _weight;
         weights[_derivativeIndex] = _weight;
-        uint256 localTotalWeight = 0;
-        for (uint256 i = 0; i < derivativeCount; i++)
-            localTotalWeight += weights[i];
-        totalWeight = localTotalWeight;
         emit WeightChange(_derivativeIndex, _weight);
     }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-L195

```solidity
File: contracts/SafEth/SafEth.sol
182:    function addDerivative(
183:        address _contractAddress,
184:        uint256 _weight
185:    ) external onlyOwner {
186:        derivatives[derivativeCount] = IDerivative(_contractAddress); 
187:        weights[derivativeCount] = _weight; 
188:        derivativeCount++; 
189:
190:        uint256 localTotalWeight = 0;
191:        for (uint256 i = 0; i < derivativeCount; i++) // @audit: sload on every iteration
192:            localTotalWeight += weights[i]; // @audit: sload on every iteration
193:        totalWeight = localTotalWeight; // @audit: localTotalWeight is totalWeight + _weight, loop is unecessary
194:        emit DerivativeAdded(_contractAddress, _weight, derivativeCount); 
195:    }
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..be44704 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -187,10 +187,7 @@ contract SafEth is
         weights[derivativeCount] = _weight;
         derivativeCount++;

-        uint256 localTotalWeight = 0;
-        for (uint256 i = 0; i < derivativeCount; i++)
-            localTotalWeight += weights[i];
-        totalWeight = localTotalWeight;
+        totalWeight = totalWeight + _weight;
         emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
     }
```

## Return values from external calls can be cached to avoid unecessary CALL
External calls are expensive as they use the `CALL` opcode (~100 gas). If you are calling the same external function more than once you should cache the return value to avoid an unecessary `CALL`.

Total Instances: `2`

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71-L75

```solidity
File: contracts/SafEth/SafEth.sol
71:        for (uint i = 0; i < derivativeCount; i++)
72:            underlyingValue +=
73:                (derivatives[i].ethPerDerivative(derivatives[i].balance()) * // @audit: 1st external call
74:                    derivatives[i].balance()) / // @audit: 2nd external call
75:                10 ** 18;
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..d5a731e 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -69,9 +69,10 @@ contract SafEth is

         // Getting underlying value in terms of ETH for each derivative
         for (uint i = 0; i < derivativeCount; i++)
+            uint256 derivativeBalance = derivatives[i].balance()
             underlyingValue +=
-                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
-                    derivatives[i].balance()) /
+                (derivatives[i].ethPerDerivative(derivativeBalance) *
+                    derivativeBalance) /
                 10 ** 18;

         uint256 totalSupply = totalSupply();
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138-L143

```solidity
File: contracts/SafEth/SafEth.sol
138:    function rebalanceToWeights() external onlyOwner {
139:        uint256 ethAmountBefore = address(this).balance;
140:        for (uint i = 0; i < derivativeCount; i++) {
141:            if (derivatives[i].balance() > 0) // @audit: 1st external call
142:                derivatives[i].withdraw(derivatives[i].balance()); // @audit: 2nd external call
143:        }
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..d5a731e 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -138,8 +139,9 @@ contract SafEth is
     function rebalanceToWeights() external onlyOwner {
         uint256 ethAmountBefore = address(this).balance;
         for (uint i = 0; i < derivativeCount; i++) {
-            if (derivatives[i].balance() > 0)
-                derivatives[i].withdraw(derivatives[i].balance());
+            uint256 derivativeBalance = derivatives[i].balance();
+            if (derivativeBalance > 0)
+                derivatives[i].withdraw(derivativeBalance);
         }
         uint256 ethAmountAfter = address(this).balance;
         uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
```

## Declare keccak256 hashes as constants to avoid computing hash each time the specified functions are called
To avoid computing string concatenation + hashing everytime the specified functions are called, we can precompute the hashes offchain and store those hashes in constant variables instead.

Total Instances: `6`

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L66-L73

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L187-L193

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L229-L235

```solidity
File: contracts/SafEth/derivatives/Reth.sol
66:    function rethAddress() private view returns (address) {
67:        return
68:            RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
69:                keccak256(
70:                    abi.encodePacked("contract.address", "rocketTokenRETH")
71:                )
72:            );
73:    }

187:            address rocketTokenRETHAddress = RocketStorageInterface(
188:                ROCKET_STORAGE_ADDRESS
189:            ).getAddress(
190:                    keccak256(
191:                        abi.encodePacked("contract.address", "rocketTokenRETH")
192:                    )
193:                );

228:    function poolPrice() private view returns (uint256) {
229:        address rocketTokenRETHAddress = RocketStorageInterface(
230:            ROCKET_STORAGE_ADDRESS
231:        ).getAddress(
232:                keccak256(
233:                    abi.encodePacked("contract.address", "rocketTokenRETH")
234:                )
235:            );
```
```diff
diff --git a/contracts/SafEth/derivatives/Reth.sol b/contracts/SafEth/derivatives/Reth.sol
index b6e0694..70fb2ad 100644
--- a/contracts/SafEth/derivatives/Reth.sol
+++ b/contracts/SafEth/derivatives/Reth.sol
@@ -25,6 +25,8 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
         0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45;
     address public constant UNI_V3_FACTORY =
         0x1F98431c8aD98523631AE4a59f267346ea31F984;
+    // keccak256(abi.encodePacked("contract.address", "rocketTokenRETH"))
+    bytes32 private constant ROCKET_TOKEN_RETH_HASH = 0xe3744443225bff7cc22028be036b80de58057d65a3fdca0a3df329f525e31ccc;

     uint256 public maxSlippage;

@@ -65,11 +67,7 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
      */
     function rethAddress() private view returns (address) {
         return
-            RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
-                keccak256(
-                    abi.encodePacked("contract.address", "rocketTokenRETH")
-                )
-            );
+            RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(ROCKET_TOKEN_RETH_HASH);
     }

     /**
@@ -186,11 +184,7 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
         } else {
             address rocketTokenRETHAddress = RocketStorageInterface(
                 ROCKET_STORAGE_ADDRESS
-            ).getAddress(
-                    keccak256(
-                        abi.encodePacked("contract.address", "rocketTokenRETH")
-                    )
-                );
+            ).getAddress(ROCKET_TOKEN_RETH_HASH);
             RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(
                 rocketTokenRETHAddress
             );
@@ -228,11 +222,7 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
     function poolPrice() private view returns (uint256) {
         address rocketTokenRETHAddress = RocketStorageInterface(
             ROCKET_STORAGE_ADDRESS
-        ).getAddress(
-                keccak256(
-                    abi.encodePacked("contract.address", "rocketTokenRETH")
-                )
-            );
+        ).getAddress(ROCKET_TOKEN_RETH_HASH);
         IUniswapV3Factory factory = IUniswapV3Factory(UNI_V3_FACTORY);
         IUniswapV3Pool pool = IUniswapV3Pool(
             factory.getPool(rocketTokenRETHAddress, W_ETH_ADDRESS, 500)
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L120-L127

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L156-L164

```solidity
File: contracts/SafEth/derivatives/Reth.sol
120:    function poolCanDeposit(uint256 _amount) private view returns (bool) {
121:        address rocketDepositPoolAddress = RocketStorageInterface(
122:            ROCKET_STORAGE_ADDRESS
123:        ).getAddress(
124:                keccak256(
125:                    abi.encodePacked("contract.address", "rocketDepositPool")
126:                )
127:            );

156:    function deposit() external payable onlyOwner returns (uint256) {
157:        // Per RocketPool Docs query addresses each time it is used
158:        address rocketDepositPoolAddress = RocketStorageInterface(
159:            ROCKET_STORAGE_ADDRESS
160:        ).getAddress(
161:                keccak256(
162:                    abi.encodePacked("contract.address", "rocketDepositPool")
163:                )
164:            );
```
```diff
diff --git a/contracts/SafEth/derivatives/Reth.sol b/contracts/SafEth/derivatives/Reth.sol
index b6e0694..024fa22 100644
--- a/contracts/SafEth/derivatives/Reth.sol
+++ b/contracts/SafEth/derivatives/Reth.sol
@@ -25,6 +25,8 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
         0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45;
     address public constant UNI_V3_FACTORY =
         0x1F98431c8aD98523631AE4a59f267346ea31F984;
+    // keccak256(abi.encodePacked("contract.address", "rocketDepositPool"))
+    bytes32 private constant ROCKET_DEPOSIT_POOL_HASH = 0x65dd923ddfc8d8ae6088f80077201d2403cbd565f0ba25e09841e2799ec90bb2;

     uint256 public maxSlippage;

@@ -120,11 +122,7 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
     function poolCanDeposit(uint256 _amount) private view returns (bool) {
         address rocketDepositPoolAddress = RocketStorageInterface(
             ROCKET_STORAGE_ADDRESS
-        ).getAddress(
-                keccak256(
-                    abi.encodePacked("contract.address", "rocketDepositPool")
-                )
-            );
+        ).getAddress(ROCKET_DEPOSIT_POOL_HASH);
         RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(
                 rocketDepositPoolAddress
             );
@@ -157,11 +155,7 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
         // Per RocketPool Docs query addresses each time it is used
         address rocketDepositPoolAddress = RocketStorageInterface(
             ROCKET_STORAGE_ADDRESS
-        ).getAddress(
-                keccak256(
-                    abi.encodePacked("contract.address", "rocketDepositPool")
-                )
-            );
+        ).getAddress(ROCKET_DEPOSIT_POOL_HASH);

         RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(
                 rocketDepositPoolAddress
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L132-L141

```solidity
File: contracts/SafEth/derivatives/Reth.sol
132:        address rocketProtocolSettingsAddress = RocketStorageInterface(
133:            ROCKET_STORAGE_ADDRESS
134:        ).getAddress(
135:                keccak256(
136:                    abi.encodePacked(
137:                        "contract.address",
138:                        "rocketDAOProtocolSettingsDeposit"
139:                    )
140:                )
141:            );
```
```diff
diff --git a/contracts/SafEth/derivatives/Reth.sol b/contracts/SafEth/derivatives/Reth.sol
index b6e0694..6c9a499 100644
--- a/contracts/SafEth/derivatives/Reth.sol
+++ b/contracts/SafEth/derivatives/Reth.sol
@@ -25,6 +25,8 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
         0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45;
     address public constant UNI_V3_FACTORY =
         0x1F98431c8aD98523631AE4a59f267346ea31F984;
+    // keccak256(abi.encodePacked("contract.address", "rocketDAOProtocolSettingsDeposit"))
+    bytes32 constant private ROCKET_SETTINGS_DEPOSIT_HASH = 0x876d8a498b5341a6b5897a8239bfb0347e2b8d81fe9401d355c1e4b0ecebedaa;

     uint256 public maxSlippage;

@@ -131,14 +133,7 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {

         address rocketProtocolSettingsAddress = RocketStorageInterface(
             ROCKET_STORAGE_ADDRESS
-        ).getAddress(
-                keccak256(
-                    abi.encodePacked(
-                        "contract.address",
-                        "rocketDAOProtocolSettingsDeposit"
-                    )
-                )
-            );
+        ).getAddress(ROCKET_SETTINGS_DEPOSIT_HASH);
         RocketDAOProtocolSettingsDepositInterface rocketDAOProtocolSettingsDeposit = RocketDAOProtocolSettingsDepositInterface(
                 rocketProtocolSettingsAddress
             );
```

## Expressions can be unchecked if gauranteed to not underflow due to previous require statement
Wrapping expressions in `unchecked` blocks will tell the solidity compiler to exclude the extra opcodes needed to check for underflows.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L200-L201

```solidity
File: contracts/SafEth/derivatives/Reth.sol
200:            require(rethBalance2 > rethBalance1, "No rETH was minted");
201:            uint256 rethMinted = rethBalance2 - rethBalance1;
```
```diff
diff --git a/contracts/SafEth/derivatives/Reth.sol b/contracts/SafEth/derivatives/Reth.sol
index b6e0694..76471b0 100644
--- a/contracts/SafEth/derivatives/Reth.sol
+++ b/contracts/SafEth/derivatives/Reth.sol
@@ -198,7 +198,10 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
             rocketDepositPool.deposit{value: msg.value}();
             uint256 rethBalance2 = rocketTokenRETH.balanceOf(address(this));
             require(rethBalance2 > rethBalance1, "No rETH was minted");
-            uint256 rethMinted = rethBalance2 - rethBalance1;
+            uint256 rethMinted;
+            unchecked {
+                rethMinted = rethBalance2 - rethBalance1;
+            }
             return (rethMinted);
         }
     }
```

## Use assembly for value transfer and success check
If we are not using the return data from a low level `.call` we can avoid creating extra stack variables and storing return data into memory by using assembly to access the `CALL` opcode and specify `0` for both the return data offset and the return data size.

Total Instances: `5`

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L63-L66

```solidity
File: contracts/SafEth/derivatives/WstEth.sol
63:        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
64:            ""
65:        );
66:        require(sent, "Failed to send Ether");
```
```diff
diff --git a/contracts/SafEth/derivatives/WstEth.sol b/contracts/SafEth/derivatives/WstEth.sol
index 5838706..95186d9 100644
--- a/contracts/SafEth/derivatives/WstEth.sol
+++ b/contracts/SafEth/derivatives/WstEth.sol
@@ -60,10 +60,12 @@ contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
         uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
         IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);
         // solhint-disable-next-line
-        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
-            ""
-        );
-        require(sent, "Failed to send Ether");
+        assembly {
+            let sent := call(gas(), caller(), selfbalance(), 0, 0, 0, 0)
+            if iszero(sent) {
+                revert(0, 0)
+            }
+        }
     }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L76-L77

```solidity
File: contracts/SafEth/derivatives/WstEth.sol
76:        (bool sent, ) = WST_ETH.call{value: msg.value}("");
77:        require(sent, "Failed to send Ether");
```
```diff
diff --git a/contracts/SafEth/derivatives/WstEth.sol b/contracts/SafEth/derivatives/WstEth.sol
index 5838706..95186d9 100644
--- a/contracts/SafEth/derivatives/WstEth.sol
+++ b/contracts/SafEth/derivatives/WstEth.sol
@@ -73,8 +75,12 @@ contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
     function deposit() external payable onlyOwner returns (uint256) {
         uint256 wstEthBalancePre = IWStETH(WST_ETH).balanceOf(address(this));
         // solhint-disable-next-line
-        (bool sent, ) = WST_ETH.call{value: msg.value}("");
-        require(sent, "Failed to send Ether");
+        assembly {
+            let sent := call(gas(), WST_ETH, callvalue(), 0, 0, 0, 0)
+            if iszero(sent) {
+                revert(0, 0)
+            }
+        }
         uint256 wstEthBalancePost = IWStETH(WST_ETH).balanceOf(address(this));
         uint256 wstEthAmount = wstEthBalancePost - wstEthBalancePre;
         return (wstEthAmount);
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L84-L87

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol
84:        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
85;            ""
86:        );
87:        require(sent, "Failed to send Ether");
```
```diff
diff --git a/contracts/SafEth/derivatives/SfrxEth.sol b/contracts/SafEth/derivatives/SfrxEth.sol
index 98a89bb..7f985da 100644
--- a/contracts/SafEth/derivatives/SfrxEth.sol
+++ b/contracts/SafEth/derivatives/SfrxEth.sol
@@ -81,10 +81,12 @@ contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {
             minOut
         );
         // solhint-disable-next-line
-        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
-            ""
-        );
-        require(sent, "Failed to send Ether");
+        assembly {
+            let sent := call(gas(), caller(), selfbalance(), 0, 0, 0, 0)
+            if iszero(sent) {
+                revert(0, 0)
+            }
+        }
     }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L110-L113

```solidity
File: contracts/SafEth/derivatives/Reth.sol
110:        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
111:            ""
112:        );
113:        require(sent, "Failed to send Ether");
```
```diff
diff --git a/contracts/SafEth/derivatives/Reth.sol b/contracts/SafEth/derivatives/Reth.sol
index b6e0694..01af157 100644
--- a/contracts/SafEth/derivatives/Reth.sol
+++ b/contracts/SafEth/derivatives/Reth.sol
@@ -107,10 +107,12 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
     function withdraw(uint256 amount) external onlyOwner {
         RocketTokenRETHInterface(rethAddress()).burn(amount);
         // solhint-disable-next-line
-        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
-            ""
-        );
-        require(sent, "Failed to send Ether");
+        assembly {
+            let sent := call(gas(), caller(), selfbalance(), 0, 0, 0, 0)
+            if iszero(sent) {
+                revert(0, 0)
+            }
+        }
     }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L124-L127

```solidity
File: contracts/SafEth/SafEth.sol
124:        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
125:            ""
126:        );
127:        require(sent, "Failed to send Ether");
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..86fa2e8 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -121,10 +121,12 @@ contract SafEth is
         uint256 ethAmountAfter = address(this).balance;
         uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;
         // solhint-disable-next-line
-        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
-            ""
-        );
-        require(sent, "Failed to send Ether");
+        assembly {
+            let sent := call(gas(), caller(), ethAmountToWithdraw, 0, 0, 0, 0)
+            if iszero(sent) {
+                revert(0, 0)
+            }
+        }
         emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
     }
```

## Use assembly to emit events
Given that the events in `SafEth.sol` have at most 2 un-indexed parameters, we are able to safetly use assembly to emit the events by using scratch space (memory locations 0x00-0x40) instead of having to handle the free memory pointer. This allows us to emit the events efficiently using a minimum amount of opcodes.

Total Instances: `10`

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L100

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L128

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L154

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L174

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L194

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L207

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L216

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L225

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L234

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L243

```solidity
File: contracts/SafEth/SafEth.sol
100:        emit Staked(msg.sender, msg.value, mintAmount);

128:        emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);

154:        emit Rebalanced();

174:        emit WeightChange(_derivativeIndex, _weight);

194:        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);

207:        emit SetMaxSlippage(_derivativeIndex, _slippage);

216:        emit ChangeMinAmount(minAmount);

225:        emit ChangeMaxAmount(maxAmount);

234:        emit StakingPaused(pauseStaking);

243:        emit UnstakingPaused(pauseUnstaking);
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..1fbc517 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -32,6 +32,27 @@ contract SafEth is
         uint index
     );
     event Rebalanced();
+
+    // keccak256("Staked(address,uint256,uint256)")
+    bytes32 private constant STAKED_EVENT_HASH = 0x1449c6dd7851abc30abf37f57715f492010519147cc2652fbc38202c18a6ee90;
+    // keccak256("Unstaked(address,uint256,uint256)")
+    bytes32 private constant UNSTAKED_EVENT_HASH = 0x7fc4727e062e336010f2c282598ef5f14facb3de68cf8195c2f23e1454b2b74e;
+    // keccak256("Rebalanced()")
+    bytes32 private constant REBALANCED_EVENT_HASH = 0xc741dbaad15a4f298fe8d80943fa8e005e7bcb2f5b0a0c8dec1fc35be457f146;
+    // keccak256("WeightChange(uint256,uint256)")
+    bytes32 private constant WEIGHT_CHANGE_EVENT_HASH = 0xaf36c6f4d67b41fb74b801bfd0a1f6e579c68ff0cc464663a49f3c6112d9b93d;
+    // keccak256("DerivativeAdded(address,uint256,uint256)")
+    bytes32 private constant DERIVATIVE_ADDED_EVENT_HASH = 0x7801a492c794d930101c8993235226b79be1216f9855e30c0615d22fdd22bb44;
+    // keccak256("SetMaxSlippage(uint256,uint256)")
+    bytes32 private constant SET_MAX_SLIPPAGE_EVENT_HASH = 0x69bd9725c402dd1e5e9d8a2323132cb3943d9dc10cb4747a89e05cda452bfe9f;
+    // keccak256("ChangeMinAmount(uint256)")
+    bytes32 private constant CHANGE_MIN_AMOUNT_EVENT_HASH = 0xbb1711a6693c8a2dfb14b13f0a1468cb1042b91bfe4e3a4b3e3d280aa255ece2;
+    // keccak256("ChangeMaxAmount(uint256)")
+    bytes32 private constant CHANGE_MAX_AMOUNT_EVENT_HASH = 0x13cad1d1f9bc3464ddf35ddde5e0389a9edf9639f97e2fccb7e59fdeb5dcad22;
+    // keccak256("StakingPaused(bool)")
+    bytes32 private constant STAKING_PAUSED_EVENT_HASH = 0x3cf4fe733160f8f8c48336fab32acee32ebaaa423a08a2be4e5ceff97dd98c69;
+    // keccak256("UnstakingPaused(bool)")
+    bytes32 private constant UNSTAKING_PAUSED_EVENT_HASH = 0x0c7f3357f9df485f4d8a04f45e87783b560f907fc7d2f72b4377ca33aa49308e;

     // As recommended by https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable
     /// @custom:oz-upgrades-unsafe-allow constructor
@@ -97,7 +118,12 @@ contract SafEth is
         // mintAmount represents a percentage of the total assets in the system
         uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
         _mint(msg.sender, mintAmount);
-        emit Staked(msg.sender, msg.value, mintAmount);
+        assembly {
+            mstore(0x00, callvalue())
+            mstore(0x20, mintAmount)
+            log2(0x00, 0x40, STAKED_EVENT_HASH, caller())
+
+        }
     }

     /**
@@ -125,7 +151,11 @@ contract SafEth is
             ""
         );
         require(sent, "Failed to send Ether");
-        emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
+        assembly {
+            mstore(0x00, ethAmountToWithdraw)
+            mstore(0x20, _safEthAmount)
+            log2(0x00, 0x40, UNSTAKED_EVENT_HASH, caller())
+        }
     }

     /**
@@ -151,7 +181,9 @@ contract SafEth is
             // Price will change due to slippage
             derivatives[i].deposit{value: ethAmount}();
         }
-        emit Rebalanced();
+        assembly {
+            log1(0x00, 0x00, REBALANCED_EVENT_HASH)
+        }
     }

     /**
@@ -171,7 +203,10 @@ contract SafEth is
         for (uint256 i = 0; i < derivativeCount; i++)
             localTotalWeight += weights[i];
         totalWeight = localTotalWeight;
-        emit WeightChange(_derivativeIndex, _weight);
+        assembly {
+            mstore(0x00, _weight)
+            log2(0x00, 0x20, WEIGHT_CHANGE_EVENT_HASH, _derivativeIndex)
+        }
     }

     /**
@@ -191,7 +226,11 @@ contract SafEth is
         for (uint256 i = 0; i < derivativeCount; i++)
             localTotalWeight += weights[i];
         totalWeight = localTotalWeight;
-        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
+        assembly {
+            mstore(0x00, _weight)
+            mstore(0x20, sload(derivativeCount.slot))
+            log2(0x00, 0x40, DERIVATIVE_ADDED_EVENT_HASH, _contractAddress)
+        }
     }

     /**
@@ -204,7 +243,10 @@ contract SafEth is
         uint _slippage
     ) external onlyOwner {
         derivatives[_derivativeIndex].setMaxSlippage(_slippage);
-        emit SetMaxSlippage(_derivativeIndex, _slippage);
+        assembly {
+            mstore(0x00, _slippage)
+            log2(0x00, 0x20, SET_MAX_SLIPPAGE_EVENT_HASH, _derivativeIndex)
+        }
     }

     /**
@@ -213,7 +255,9 @@ contract SafEth is
     */
     function setMinAmount(uint256 _minAmount) external onlyOwner {
         minAmount = _minAmount;
-        emit ChangeMinAmount(minAmount);
+        assembly {
+            log2(0x00, 0x00, CHANGE_MIN_AMOUNT_EVENT_HASH, calldataload(0x04))
+        }
     }

     /**
@@ -222,7 +266,9 @@ contract SafEth is
     */
     function setMaxAmount(uint256 _maxAmount) external onlyOwner {
         maxAmount = _maxAmount;
-        emit ChangeMaxAmount(maxAmount);
+        assembly {
+            log2(0x00, 0x00, CHANGE_MAX_AMOUNT_EVENT_HASH, calldataload(0x04))
+        }
     }

     /**
@@ -231,7 +277,9 @@ contract SafEth is
     */
     function setPauseStaking(bool _pause) external onlyOwner {
         pauseStaking = _pause;
-        emit StakingPaused(pauseStaking);
+        assembly {
+            log2(0x00, 0x00, STAKING_PAUSED_EVENT_HASH, calldataload(0x04))
+        }
     }

     /**
@@ -240,8 +288,11 @@ contract SafEth is
     */
     function setPauseUnstaking(bool _pause) external onlyOwner {
         pauseUnstaking = _pause;
-        emit UnstakingPaused(pauseUnstaking);
+        assembly {
+            log2(0x00, 0x00, UNSTAKING_PAUSED_EVENT_HASH, calldataload(0x04))
+        }
     }
```

## Use assembly for loops
In the following instances assembly is used for more gas efficient for loops. To avoid mishandling the free memory pointer, only scratch space is used for all necessary operations that involve memory (i.e. hashing, storing return data, and storing function parameters). In addition, since instances include multiple external calls we can optimize for this by loading both function signatures into memory as one word. The individual function signatures are then accessed for each call by specifying their offset in memory.

Total Instances: `3`

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71-L75

```solidity
File: contracts/SafEth/SafEth.sol
71:        for (uint i = 0; i < derivativeCount; i++)
72:            underlyingValue +=
73:                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74:                    derivatives[i].balance()) /
75:                10 ** 18;
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..e09841c 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -68,11 +68,26 @@ contract SafEth is
         uint256 underlyingValue = 0;

         // Getting underlying value in terms of ETH for each derivative
-        for (uint i = 0; i < derivativeCount; i++)
-            underlyingValue +=
-                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
-                    derivatives[i].balance()) /
-                10 ** 18;
+        assembly {
+            let _derivativeCount := sload(derivativeCount.slot)
+            for { let i := 0 } lt(i, _derivativeCount) { i := add(i, 1) } {
+                mstore(0x00, i) // key for mapping
+                mstore(0x20, derivatives.slot) // slot for mapping
+                let derivative := sload(keccak256(0x00, 0x40)) // storage location for mapping value is keccak256(key, slot)
+                mstore(0x00, 0xb69ef8a82bd6de1d) // function signatures for "balance()" + "ethPerDerivative(uint256)"
+                let success1 := staticcall(gas(), derivative, 0x18, 0x04, 0x20, 0x20) // 0x18 == offset for "balance()"
+                if iszero(success1) {
+                    revert(0, 0)
+                }
+                let bal := mload(0x20) // load from return data from first call
+                let success2 := staticcall(gas(), derivative, 0x1c, 0x24, 0x20, 0x20) // 0x1c == offset for "ethPerDerivative(uint256)"
+                if iszero(success2) {
+                    revert(0, 0)
+                }
+                let eth := mload(0x20) // load return data from second call
+                underlyingValue := add(underlyingValue, div(mul(eth, bal), 0xDE0B6B3A7640000)) // 0xDE0B6B3A7640000 == 10 ** 18
+            }
+        }

         uint256 totalSupply = totalSupply();
         uint256 preDepositPrice; // Price of safETH in regards to ETH
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84-L96

```solidity
File: contracts/SafEth/SafEth.sol
84:        for (uint i = 0; i < derivativeCount; i++) {
83:            uint256 weight = weights[i];
84:            IDerivative derivative = derivatives[i];
85:            if (weight == 0) continue;
86:            uint256 ethAmount = (msg.value * weight) / totalWeight;
87:
88:            // This is slightly less than ethAmount because slippage
89:            uint256 depositAmount = derivative.deposit{value: ethAmount}();
90:            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
91:                depositAmount
92:            ) * depositAmount) / 10 ** 18;
93:            totalStakeValueEth += derivativeReceivedEthValue;
94:        }
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..e09841c 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -81,18 +96,32 @@ contract SafEth is
         else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

         uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
-        for (uint i = 0; i < derivativeCount; i++) {
-            uint256 weight = weights[i];
-            IDerivative derivative = derivatives[i];
-            if (weight == 0) continue;
-            uint256 ethAmount = (msg.value * weight) / totalWeight;
-
-            // This is slightly less than ethAmount because slippage
-            uint256 depositAmount = derivative.deposit{value: ethAmount}();
-            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
-                depositAmount
-            ) * depositAmount) / 10 ** 18;
-            totalStakeValueEth += derivativeReceivedEthValue;
+        assembly {
+            let _derivativeCount := sload(derivativeCount.slot)
+            let _totalWeight := sload(totalWeight.slot)
+            for { let i := 0 } lt(i, _derivativeCount) { i := add(i, 1) } {
+                mstore(0x00, i) // key for mapping
+                mstore(0x20, weights.slot) // slot for weights mapping 
+                let weight := sload(keccak256(0x00, 0x40)) // storage location for mapping value is keccak256(Key, Slot)
+                if iszero(iszero(weight)) { // "if weight != 0"
+                    mstore(0x20, derivatives.slot) // slot for derivatives mapping
+                    let derivative := sload(keccak256(0x00, 0x40)) // key for mapping was previously copied into mem at offset 0x00
+                    let ethAmount := div(mul(callvalue(), weight), _totalWeight)
+
+                    mstore(0x00, 0xd0e30db02bd6de1d) // function signatures for "deposit()" + "ethPerDerivative(uint256)"
+                    let success1 := call(gas(), derivative, ethAmount, 0x18, 0x04, 0x20, 0x20) // 0x18 == offset for "deposit()"
+                    if iszero(success1) {
+                        revert(0, 0)
+                    }
+                    let depositAmount := mload(0x20) // return data from first call
+                    let success2 := staticcall(gas(), derivative, 0x1c, 0x24, 0x20, 0x20) // 0x1c == offset for "ethPerDerivative(uint256)"
+                    if iszero(success2) {
+                        revert(0, 0)
+                    }
+                    let eth := mload(0x20) // return data from second call
+                    totalStakeValueEth := add(totalStakeValueEth, div(mul(eth, depositAmount), 0xDE0B6B3A7640000)) // 0xDE0B6B3A7640000 == 10 **18
+                }
+            }
         }
         // mintAmount represents a percentage of the total assets in the system
         uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113-L119

```solidity
File: contracts/SafEth/SafEth.sol
113:        for (uint256 i = 0; i < derivativeCount; i++) {
114:            // withdraw a percentage of each asset based on the amount of safETH
115:            uint256 derivativeAmount = (derivatives[i].balance() *
116:                _safEthAmount) / safEthTotalSupply;
117:            if (derivativeAmount == 0) continue; // if derivative empty ignore
118:            derivatives[i].withdraw(derivativeAmount);
119:        }
```
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..e09841c 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -110,12 +139,26 @@ contract SafEth is
         uint256 safEthTotalSupply = totalSupply();
         uint256 ethAmountBefore = address(this).balance;

-        for (uint256 i = 0; i < derivativeCount; i++) {
-            // withdraw a percentage of each asset based on the amount of safETH
-            uint256 derivativeAmount = (derivatives[i].balance() *
-                _safEthAmount) / safEthTotalSupply;
-            if (derivativeAmount == 0) continue; // if derivative empty ignore
-            derivatives[i].withdraw(derivativeAmount);
+        assembly {
+            let _derivativeCount := sload(derivativeCount.slot)
+            for { let i := 0 } lt(i, _derivativeCount) { i := add(i, 1) } {
+                mstore(0x00, i) // key for mapping
+                mstore(0x20, derivatives.slot) // slot for mapping
+                let derivative := sload(keccak256(0x00, 0x40)) // storage location for mapping value is keccak256(Key, Slot)
+                mstore(0x00, 0xb69ef8a82e1a7d4d) // function signatures for "balance()" and "withdraw(uint256)"
+                let success1 := staticcall(gas(), derivative, 0x18, 0x04, 0x20, 0x20) // 0x18 == offset for "balance()"
+                if iszero(success1) {
+                    revert(0, 0)
+                }
+                let derivativeAmount := div(mul(mload(0x20), calldataload(0x04)), safEthTotalSupply)
+                if iszero(iszero(derivativeAmount)) { // "if derivativeAmount != 0"
+                    mstore(0x20, derivativeAmount)
+                    let success2 := call(gas(), derivative, 0x00, 0x1c, 0x24, 0x00, 0x00) // 0x1c == offset for "withdraw(uint256)"
+                    if iszero(success2) {
+                        revert(0, 0)
+                    }
+                }
+            }
         }
         _burn(msg.sender, _safEthAmount);
         uint256 ethAmountAfter = address(this).balance;
```

## Adding a derivative will increase gas costs for users staking and unstaking even after the derivative is removed
When a derivative is added, `derivativeCount` is incremented. This results in more loop iterations in `stake()` and `unstake()`, which results in a total of 11 more SLOADs and 7 more external calls between `stake()` and `unstake()` per iteration. According to [@0xToshi](https://discord.com/channels/810916927919620096/1088690534735945789/1089630285521490011), derivatives can be removed by setting their `weight` to 0. However, since the `derivativeCount` storage variable is not decremented during this update, the users will still need to pay for those extra SLOADs and external calls since `derivativeCount` (which dictates the number of loop iterations) remains the same.

A potential solution for this issue would be to explicitly remove the derivative by setting `derivative[index] = address(0)` in either an existing privileged function or its own `removeDerivative()` function. Now, we can perform a similar check as seen [here](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L87) in all for loops to ignore derivatives that are removed: `if (address(derivatives[i]) == address(0)) continue;`. This will result in us avoiding unecessary SLOADs and external calls when `derivatives[i]` is equal to a removed derivative. 

**Note that in this solution `derivativeCount` remains the same. This is because decrementing `derivativeCount` will effectively remove the last added derivative from the scopes of the for loops, not the actual removed derivative (if we are removing a derivative with `index != derivativeCount`). Therefore, the for loops will still have "`derivativeCount`" number of iterations, but SLOADs and external calls will be saved when iterating over a removed derivative.**

### Below is a diff illustrating the additional check in the for loops (assuming the removal logic is implemented elsewhere):
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..b1609a7 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -68,11 +68,13 @@ contract SafEth is
         uint256 underlyingValue = 0;

         // Getting underlying value in terms of ETH for each derivative
-        for (uint i = 0; i < derivativeCount; i++)
+        for (uint i = 0; i < derivativeCount; i++) {
+            if (address(derivatives[i]) == address(0)) continue;
             underlyingValue +=
                 (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                     derivatives[i].balance()) /
                 10 ** 18;
+        }

         uint256 totalSupply = totalSupply();
         uint256 preDepositPrice; // Price of safETH in regards to ETH
@@ -84,7 +86,7 @@ contract SafEth is
         for (uint i = 0; i < derivativeCount; i++) {
             uint256 weight = weights[i];
             IDerivative derivative = derivatives[i];
-            if (weight == 0) continue;
+            if (weight == 0 || address(derivatives[i]) == address(0)) continue;
             uint256 ethAmount = (msg.value * weight) / totalWeight;

             // This is slightly less than ethAmount because slippage
@@ -111,6 +113,7 @@ contract SafEth is
         uint256 ethAmountBefore = address(this).balance;

         for (uint256 i = 0; i < derivativeCount; i++) {
+            if (address(derivatives[i]) == address(0)) continue;
             // withdraw a percentage of each asset based on the amount of safETH
             uint256 derivativeAmount = (derivatives[i].balance() *
                 _safEthAmount) / safEthTotalSupply;
@@ -138,6 +141,7 @@ contract SafEth is
     function rebalanceToWeights() external onlyOwner {
         uint256 ethAmountBefore = address(this).balance;
         for (uint i = 0; i < derivativeCount; i++) {
+            if (address(derivatives[i]) == address(0)) continue;
             if (derivatives[i].balance() > 0)
                 derivatives[i].withdraw(derivatives[i].balance());
         }
@@ -145,7 +149,7 @@ contract SafEth is
         uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;

         for (uint i = 0; i < derivativeCount; i++) {
-            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
+            if (weights[i] == 0 || address(derivatives[i]) == address(0) || ethAmountToRebalance == 0) continue;
             uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                 totalWeight;
             // Price will change due to slippage
```

## GasReporter Output with all optimizations applied

```js
·------------------------------------------------|---------------------------|----------------|-----------------------------·
|              Solc version: 0.8.13              ·  Optimizer enabled: true  ·  Runs: 100000  ·  Block limit: 30000000 gas  │
·················································|···························|················|······························
|  Methods                                                                                                                  │
·····················|···························|·············|·············|················|···············|··············
|  Contract          ·  Method                   ·  Min        ·  Max        ·  Avg           ·  # calls      ·  usd (avg)  │
·····················|···························|·············|·············|················|···············|··············
|  DerivativeMock    ·  withdrawAll              ·          -  ·          -  ·        164908  ·            2  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  ERC20Upgradeable  ·  transfer                 ·          -  ·          -  ·         51712  ·            1  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  Reth              ·  deposit                  ·     140315  ·     230700  ·        176124  ·           10  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  Reth              ·  withdraw                 ·     144399  ·     224388  ·        180930  ·            6  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  SafEth            ·  addDerivative            ·      87720  ·     121920  ·         99120  ·           21  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  SafEth            ·  adjustWeight             ·      37287  ·      59987  ·         39195  ·           35  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  SafEth            ·  rebalanceToWeights       ·     659055  ·     810137  ·        717607  ·            8  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  SafEth            ·  setMaxAmount             ·          -  ·          -  ·         37146  ·            1  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  SafEth            ·  setMaxSlippage           ·      47512  ·      69538  ·         58532  ·            6  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  SafEth            ·  setMinAmount             ·          -  ·          -  ·         37090  ·            1  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  SafEth            ·  setPauseStaking          ·          -  ·          -  ·         54263  ·            2  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  SafEth            ·  setPauseUnstaking        ·          -  ·          -  ·         37193  ·            2  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  SafEth            ·  stake                    ·     423368  ·     644102  ·        511519  ·          355  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  SafEth            ·  unstake                  ·     415756  ·     567688  ·        513131  ·          386  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  SafEthV2Mock      ·  adminWithdrawDerivative  ·     160103  ·     240104  ·        195594  ·            6  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  SafEthV2Mock      ·  adminWithdrawErc20       ·          -  ·          -  ·         59748  ·            2  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  SafEthV2Mock      ·  newFunction              ·          -  ·          -  ·         50743  ·            4  ·          -  │
·····················|···························|·············|·············|················|···············|··············
|  Deployments                                   ·                                            ·  % of limit   ·             │
·················································|·············|·············|················|···············|··············
|  DerivativeMock                                ·          -  ·          -  ·       1107926  ·        3.7 %  ·          -  │
·················································|·············|·············|················|···············|··············
|  Reth                                          ·          -  ·          -  ·       1398415  ·        4.7 %  ·          -  │
·················································|·············|·············|················|···············|··············
|  SafEth                                        ·          -  ·          -  ·       2085496  ·          7 %  ·          -  │
·················································|·············|·············|················|···············|··············
|  SafEthV2Mock                                  ·          -  ·          -  ·       2217981  ·        7.4 %  ·          -  │
·················································|·············|·············|················|···············|··············
|  SfrxEth                                       ·          -  ·          -  ·        921426  ·        3.1 %  ·          -  │
·················································|·············|·············|················|···············|··············
|  WstEth                                        ·          -  ·          -  ·        838141  ·        2.8 %  ·          -  │
·------------------------------------------------|-------------|-------------|----------------|---------------|-------------·
```
## Diff for SafEth.sol with all optimizations applied
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..e09841c 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -32,6 +32,27 @@ contract SafEth is
         uint index
     );
     event Rebalanced();
+
+    // keccak256("Staked(address,uint256,uint256)")
+    bytes32 private constant STAKED_EVENT_HASH = 0x1449c6dd7851abc30abf37f57715f492010519147cc2652fbc38202c18a6ee90;
+    // keccak256("Unstaked(address,uint256,uint256)")
+    bytes32 private constant UNSTAKED_EVENT_HASH = 0x7fc4727e062e336010f2c282598ef5f14facb3de68cf8195c2f23e1454b2b74e;
+    // keccak256("Rebalanced()")
+    bytes32 private constant REBALANCED_EVENT_HASH = 0xc741dbaad15a4f298fe8d80943fa8e005e7bcb2f5b0a0c8dec1fc35be457f146;
+    // keccak256("WeightChange(uint256,uint256)")
+    bytes32 private constant WEIGHT_CHANGE_EVENT_HASH = 0xaf36c6f4d67b41fb74b801bfd0a1f6e579c68ff0cc464663a49f3c6112d9b93d;
+    // keccak256("DerivativeAdded(address,uint256,uint256)")
+    bytes32 private constant DERIVATIVE_ADDED_EVENT_HASH = 0x7801a492c794d930101c8993235226b79be1216f9855e30c0615d22fdd22bb44;
+    // keccak256("SetMaxSlippage(uint256,uint256)")
+    bytes32 private constant SET_MAX_SLIPPAGE_EVENT_HASH = 0x69bd9725c402dd1e5e9d8a2323132cb3943d9dc10cb4747a89e05cda452bfe9f;
+    // keccak256("ChangeMinAmount(uint256)")
+    bytes32 private constant CHANGE_MIN_AMOUNT_EVENT_HASH = 0xbb1711a6693c8a2dfb14b13f0a1468cb1042b91bfe4e3a4b3e3d280aa255ece2;
+    // keccak256("ChangeMaxAmount(uint256)")
+    bytes32 private constant CHANGE_MAX_AMOUNT_EVENT_HASH = 0x13cad1d1f9bc3464ddf35ddde5e0389a9edf9639f97e2fccb7e59fdeb5dcad22;
+    // keccak256("StakingPaused(bool)")
+    bytes32 private constant STAKING_PAUSED_EVENT_HASH = 0x3cf4fe733160f8f8c48336fab32acee32ebaaa423a08a2be4e5ceff97dd98c69;
+    // keccak256("UnstakingPaused(bool)")
+    bytes32 private constant UNSTAKING_PAUSED_EVENT_HASH = 0x0c7f3357f9df485f4d8a04f45e87783b560f907fc7d2f72b4377ca33aa49308e;

     // As recommended by https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable
     /// @custom:oz-upgrades-unsafe-allow constructor
@@ -68,11 +89,26 @@ contract SafEth is
         uint256 underlyingValue = 0;

         // Getting underlying value in terms of ETH for each derivative
-        for (uint i = 0; i < derivativeCount; i++)
-            underlyingValue +=
-                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
-                    derivatives[i].balance()) /
-                10 ** 18;
+        uint256 _derivativeCount = derivativeCount;
+        assembly {
+            for { let i := 0 } lt(i, _derivativeCount) { i := add(i, 1) } {
+                mstore(0x00, i)
+                mstore(0x20, derivatives.slot)
+                let derivative := sload(keccak256(0x00, 0x40))
+                mstore(0x00, 0xb69ef8a82bd6de1d)
+                let success1 := staticcall(gas(), derivative, 0x18, 0x04, 0x20, 0x20)
+                if iszero(success1) {
+                    revert(0, 0)
+                }
+                let bal := mload(0x20)
+                let success2 := staticcall(gas(), derivative, 0x1c, 0x24, 0x20, 0x20)
+                if iszero(success2) {
+                    revert(0, 0)
+                }
+                let eth := mload(0x20)
+                underlyingValue := add(underlyingValue, div(mul(eth, bal), 0xDE0B6B3A7640000))
+            }
+        }

         uint256 totalSupply = totalSupply();
         uint256 preDepositPrice; // Price of safETH in regards to ETH
@@ -81,23 +117,41 @@ contract SafEth is
         else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

         uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
-        for (uint i = 0; i < derivativeCount; i++) {
-            uint256 weight = weights[i];
-            IDerivative derivative = derivatives[i];
-            if (weight == 0) continue;
-            uint256 ethAmount = (msg.value * weight) / totalWeight;
-
-            // This is slightly less than ethAmount because slippage
-            uint256 depositAmount = derivative.deposit{value: ethAmount}();
-            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
-                depositAmount
-            ) * depositAmount) / 10 ** 18;
-            totalStakeValueEth += derivativeReceivedEthValue;
+        uint256 _totalWeight = totalWeight;
+        assembly {
+            for { let i := 0 } lt(i, _derivativeCount) { i := add(i, 1) } {
+                mstore(0x00, i)
+                mstore(0x20, weights.slot)
+                let weight := sload(keccak256(0x00, 0x40))
+                if iszero(iszero(weight)) {
+                    mstore(0x20, derivatives.slot)
+                    let derivative := sload(keccak256(0x00, 0x40))
+                    let ethAmount := div(mul(callvalue(), weight), _totalWeight)
+
+                    mstore(0x00, 0xd0e30db02bd6de1d)
+                    let success1 := call(gas(), derivative, ethAmount, 0x18, 0x04, 0x20, 0x20)
+                    if iszero(success1) {
+                        revert(0, 0)
+                    }
+                    let depositAmount := mload(0x20)
+                    let success2 := staticcall(gas(), derivative, 0x1c, 0x24, 0x20, 0x20)
+                    if iszero(success2) {
+                        revert(0, 0)
+                    }
+                    let eth := mload(0x20)
+                    totalStakeValueEth := add(totalStakeValueEth, div(mul(eth, depositAmount), 0xDE0B6B3A7640000))
+                }
+            }
         }
         // mintAmount represents a percentage of the total assets in the system
         uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
         _mint(msg.sender, mintAmount);
-        emit Staked(msg.sender, msg.value, mintAmount);
+        assembly {
+            mstore(0x00, callvalue())
+            mstore(0x20, mintAmount)
+            log2(0x00, 0x40, STAKED_EVENT_HASH, caller())
+
+        }
     }

     /**
@@ -109,23 +163,42 @@ contract SafEth is
         require(pauseUnstaking == false, "unstaking is paused");
         uint256 safEthTotalSupply = totalSupply();
         uint256 ethAmountBefore = address(this).balance;
-
-        for (uint256 i = 0; i < derivativeCount; i++) {
-            // withdraw a percentage of each asset based on the amount of safETH
-            uint256 derivativeAmount = (derivatives[i].balance() *
-                _safEthAmount) / safEthTotalSupply;
-            if (derivativeAmount == 0) continue; // if derivative empty ignore
-            derivatives[i].withdraw(derivativeAmount);
+        uint256 _derivativeCount = derivativeCount;
+        assembly {
+            for { let i := 0 } lt(i, _derivativeCount) { i := add(i, 1) } {
+                mstore(0x00, i)
+                mstore(0x20, derivatives.slot)
+                let derivative := sload(keccak256(0x00, 0x40))
+                mstore(0x00, 0xb69ef8a82e1a7d4d)
+                let success1 := staticcall(gas(), derivative, 0x18, 0x04, 0x20, 0x20)
+                if iszero(success1) {
+                    revert(0, 0)
+                }
+                let derivativeAmount := div(mul(mload(0x20), calldataload(0x04)), safEthTotalSupply)
+                if iszero(iszero(derivativeAmount)) {
+                    mstore(0x20, derivativeAmount)
+                    let success2 := call(gas(), derivative, 0x00, 0x1c, 0x24, 0x00, 0x00)
+                    if iszero(success2) {
+                        revert(0, 0)
+                    }
+                }
+            }
         }
         _burn(msg.sender, _safEthAmount);
         uint256 ethAmountAfter = address(this).balance;
         uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;
         // solhint-disable-next-line
-        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
-            ""
-        );
-        require(sent, "Failed to send Ether");
-        emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
+        assembly {
+            let sent := call(gas(), caller(), ethAmountToWithdraw, 0, 0, 0, 0)
+            if iszero(sent) {
+                revert(0, 0)
+            }
+        }
+        assembly {
+            mstore(0x00, ethAmountToWithdraw)
+            mstore(0x20, _safEthAmount)
+            log2(0x00, 0x40, UNSTAKED_EVENT_HASH, caller())
+        }
     }

     /**
@@ -137,21 +210,28 @@ contract SafEth is
     */
     function rebalanceToWeights() external onlyOwner {
         uint256 ethAmountBefore = address(this).balance;
-        for (uint i = 0; i < derivativeCount; i++) {
-            if (derivatives[i].balance() > 0)
-                derivatives[i].withdraw(derivatives[i].balance());
+        uint256 _derivativeCount = derivativeCount;
+        for (uint i = 0; i < _derivativeCount; i++) {
+            IDerivative derivative = derivatives[i];
+            uint256 derivativeBalance = derivative.balance();
+            if (derivativeBalance > 0)
+                derivative.withdraw(derivativeBalance);
         }
         uint256 ethAmountAfter = address(this).balance;
         uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
+        uint256 _totalWeight = totalWeight;

-        for (uint i = 0; i < derivativeCount; i++) {
-            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
-            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
-                totalWeight;
+        for (uint i = 0; i < _derivativeCount; i++) {
+            uint256 weight = weights[i];
+            if (weight == 0 || ethAmountToRebalance == 0) continue;
+            uint256 ethAmount = (ethAmountToRebalance * weight) /
+                _totalWeight;
             // Price will change due to slippage
             derivatives[i].deposit{value: ethAmount}();
         }
-        emit Rebalanced();
+        assembly {
+            log1(0x00, 0x00, REBALANCED_EVENT_HASH)
+        }
     }

     /**
@@ -166,12 +246,13 @@ contract SafEth is
         uint256 _derivativeIndex,
         uint256 _weight
     ) external onlyOwner {
+        uint256 oldWeight = weights[_derivativeIndex];
+        totalWeight = totalWeight - oldWeight + _weight;
         weights[_derivativeIndex] = _weight;
-        uint256 localTotalWeight = 0;
-        for (uint256 i = 0; i < derivativeCount; i++)
-            localTotalWeight += weights[i];
-        totalWeight = localTotalWeight;
-        emit WeightChange(_derivativeIndex, _weight);
+        assembly {
+            mstore(0x00, _weight)
+            log2(0x00, 0x20, WEIGHT_CHANGE_EVENT_HASH, _derivativeIndex)
+        }
     }

     /**
@@ -183,15 +264,18 @@ contract SafEth is
         address _contractAddress,
         uint256 _weight
     ) external onlyOwner {
-        derivatives[derivativeCount] = IDerivative(_contractAddress);
-        weights[derivativeCount] = _weight;
-        derivativeCount++;
-
-        uint256 localTotalWeight = 0;
-        for (uint256 i = 0; i < derivativeCount; i++)
-            localTotalWeight += weights[i];
-        totalWeight = localTotalWeight;
-        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
+        uint256 _derivativeCount = derivativeCount;
+        derivatives[_derivativeCount] = IDerivative(_contractAddress);
+        weights[_derivativeCount] = _weight;
+        uint256 newDerivativeCount = _derivativeCount + 1;
+        derivativeCount = newDerivativeCount;
+
+        totalWeight = totalWeight + _weight;
+        assembly {
+            mstore(0x00, _weight)
+            mstore(0x20, newDerivativeCount)
+            log2(0x00, 0x40, DERIVATIVE_ADDED_EVENT_HASH, _contractAddress)
+        }
     }

     /**
@@ -204,7 +288,10 @@ contract SafEth is
         uint _slippage
     ) external onlyOwner {
         derivatives[_derivativeIndex].setMaxSlippage(_slippage);
-        emit SetMaxSlippage(_derivativeIndex, _slippage);
+        assembly {
+            mstore(0x00, _slippage)
+            log2(0x00, 0x20, SET_MAX_SLIPPAGE_EVENT_HASH, _derivativeIndex)
+        }
     }

     /**
@@ -213,7 +300,9 @@ contract SafEth is
     */
     function setMinAmount(uint256 _minAmount) external onlyOwner {
         minAmount = _minAmount;
-        emit ChangeMinAmount(minAmount);
+        assembly {
+            log2(0x00, 0x00, CHANGE_MIN_AMOUNT_EVENT_HASH, calldataload(0x04))
+        }
     }

     /**
@@ -222,7 +311,9 @@ contract SafEth is
     */
     function setMaxAmount(uint256 _maxAmount) external onlyOwner {
         maxAmount = _maxAmount;
-        emit ChangeMaxAmount(maxAmount);
+        assembly {
+            log2(0x00, 0x00, CHANGE_MAX_AMOUNT_EVENT_HASH, calldataload(0x04))
+        }
     }

     /**
@@ -231,7 +322,9 @@ contract SafEth is
     */
     function setPauseStaking(bool _pause) external onlyOwner {
         pauseStaking = _pause;
-        emit StakingPaused(pauseStaking);
+        assembly {
+            log2(0x00, 0x00, STAKING_PAUSED_EVENT_HASH, calldataload(0x04))
+        }
     }

     /**
@@ -240,8 +333,12 @@ contract SafEth is
     */
     function setPauseUnstaking(bool _pause) external onlyOwner {
         pauseUnstaking = _pause;
-        emit UnstakingPaused(pauseUnstaking);
+        assembly {
+            log2(0x00, 0x00, UNSTAKING_PAUSED_EVENT_HASH, calldataload(0x04))
+        }
     }
```
## Diff for Reth.sol with all optimizations applied
```diff
diff --git a/contracts/SafEth/derivatives/Reth.sol b/contracts/SafEth/derivatives/Reth.sol
index b6e0694..f749a92 100644
--- a/contracts/SafEth/derivatives/Reth.sol
+++ b/contracts/SafEth/derivatives/Reth.sol
@@ -25,6 +25,12 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
         0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45;
     address public constant UNI_V3_FACTORY =
         0x1F98431c8aD98523631AE4a59f267346ea31F984;
+    // keccak256(abi.encodePacked("contract.address", "rocketTokenRETH"))
+    bytes32 private constant ROCKET_TOKEN_RETH_HASH = 0xe3744443225bff7cc22028be036b80de58057d65a3fdca0a3df329f525e31ccc;
+    // keccak256(abi.encodePacked("contract.address", "rocketDepositPool"))
+    bytes32 private constant ROCKET_DEPOSIT_POOL_HASH = 0x65dd923ddfc8d8ae6088f80077201d2403cbd565f0ba25e09841e2799ec90bb2;
+    // keccak256(abi.encodePacked("contract.address", "rocketDAOProtocolSettingsDeposit"))
+    bytes32 constant private ROCKET_SETTINGS_DEPOSIT_HASH = 0x876d8a498b5341a6b5897a8239bfb0347e2b8d81fe9401d355c1e4b0ecebedaa;

     uint256 public maxSlippage;

@@ -64,12 +70,7 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
         @dev - per RocketPool Docs query addresses each time it is used
      */
     function rethAddress() private view returns (address) {
-        return
-            RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
-                keccak256(
-                    abi.encodePacked("contract.address", "rocketTokenRETH")
-                )
-            );
+        return RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(ROCKET_TOKEN_RETH_HASH);
     }

     /**
@@ -107,10 +108,12 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
     function withdraw(uint256 amount) external onlyOwner {
         RocketTokenRETHInterface(rethAddress()).burn(amount);
         // solhint-disable-next-line
-        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
-            ""
-        );
-        require(sent, "Failed to send Ether");
+        assembly {
+            let sent := call(gas(), caller(), selfbalance(), 0, 0, 0, 0)
+            if iszero(sent) {
+                revert(0, 0)
+            }
+        }
     }

     /**
@@ -120,25 +123,14 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
     function poolCanDeposit(uint256 _amount) private view returns (bool) {
         address rocketDepositPoolAddress = RocketStorageInterface(
             ROCKET_STORAGE_ADDRESS
-        ).getAddress(
-                keccak256(
-                    abi.encodePacked("contract.address", "rocketDepositPool")
-                )
-            );
+        ).getAddress(ROCKET_DEPOSIT_POOL_HASH);
         RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(
                 rocketDepositPoolAddress
             );

         address rocketProtocolSettingsAddress = RocketStorageInterface(
             ROCKET_STORAGE_ADDRESS
-        ).getAddress(
-                keccak256(
-                    abi.encodePacked(
-                        "contract.address",
-                        "rocketDAOProtocolSettingsDeposit"
-                    )
-                )
-            );
+        ).getAddress(ROCKET_SETTINGS_DEPOSIT_HASH);
         RocketDAOProtocolSettingsDepositInterface rocketDAOProtocolSettingsDeposit = RocketDAOProtocolSettingsDepositInterface(
                 rocketProtocolSettingsAddress
             );
@@ -157,11 +149,7 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
         // Per RocketPool Docs query addresses each time it is used
         address rocketDepositPoolAddress = RocketStorageInterface(
             ROCKET_STORAGE_ADDRESS
-        ).getAddress(
-                keccak256(
-                    abi.encodePacked("contract.address", "rocketDepositPool")
-                )
-            );
+        ).getAddress(ROCKET_DEPOSIT_POOL_HASH);

         RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(
                 rocketDepositPoolAddress
@@ -186,11 +174,7 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
         } else {
             address rocketTokenRETHAddress = RocketStorageInterface(
                 ROCKET_STORAGE_ADDRESS
-            ).getAddress(
-                    keccak256(
-                        abi.encodePacked("contract.address", "rocketTokenRETH")
-                    )
-                );
+            ).getAddress(ROCKET_TOKEN_RETH_HASH);
             RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(
                 rocketTokenRETHAddress
             );
@@ -198,7 +182,10 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
             rocketDepositPool.deposit{value: msg.value}();
             uint256 rethBalance2 = rocketTokenRETH.balanceOf(address(this));
             require(rethBalance2 > rethBalance1, "No rETH was minted");
-            uint256 rethMinted = rethBalance2 - rethBalance1;
+            uint256 rethMinted;
+            unchecked {
+                rethMinted = rethBalance2 - rethBalance1;
+            }
             return (rethMinted);
         }
     }
@@ -228,11 +215,7 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
     function poolPrice() private view returns (uint256) {
         address rocketTokenRETHAddress = RocketStorageInterface(
             ROCKET_STORAGE_ADDRESS
-        ).getAddress(
-                keccak256(
-                    abi.encodePacked("contract.address", "rocketTokenRETH")
-                )
-            );
+        ).getAddress(ROCKET_TOKEN_RETH_HASH);
         IUniswapV3Factory factory = IUniswapV3Factory(UNI_V3_FACTORY);
         IUniswapV3Pool pool = IUniswapV3Pool(
             factory.getPool(rocketTokenRETHAddress, W_ETH_ADDRESS, 500)
```
## Diff for SfrxEth.sol with all optimizations applied
```diff
diff --git a/contracts/SafEth/derivatives/SfrxEth.sol b/contracts/SafEth/derivatives/SfrxEth.sol
index 98a89bb..877e23e 100644
--- a/contracts/SafEth/derivatives/SfrxEth.sol
+++ b/contracts/SafEth/derivatives/SfrxEth.sol
@@ -81,10 +81,12 @@ contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {
             minOut
         );
         // solhint-disable-next-line
-        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
-            ""
-        );
-        require(sent, "Failed to send Ether");
+        assembly {
+            let sent := call(gas(), caller(), selfbalance(), 0, 0, 0, 0)
+            if iszero(sent) {
+                revert(0, 0)
+            }
+        }
     }
```
## Diff for WstEth.sol with all optimizations applied
```diff
diff --git a/contracts/SafEth/derivatives/WstEth.sol b/contracts/SafEth/derivatives/WstEth.sol
index 5838706..fb4c858 100644
--- a/contracts/SafEth/derivatives/WstEth.sol
+++ b/contracts/SafEth/derivatives/WstEth.sol
@@ -60,10 +60,12 @@ contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
         uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
         IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);
         // solhint-disable-next-line
-        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
-            ""
-        );
-        require(sent, "Failed to send Ether");
+        assembly {
+            let sent := call(gas(), caller(), selfbalance(), 0, 0, 0, 0)
+            if iszero(sent) {
+                revert(0, 0)
+            }
+        }
     }

     /**
@@ -73,8 +75,12 @@ contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
     function deposit() external payable onlyOwner returns (uint256) {
         uint256 wstEthBalancePre = IWStETH(WST_ETH).balanceOf(address(this));
         // solhint-disable-next-line
-        (bool sent, ) = WST_ETH.call{value: msg.value}("");
-        require(sent, "Failed to send Ether");
+        assembly {
+            let sent := call(gas(), WST_ETH, callvalue(), 0, 0, 0, 0)
+            if iszero(sent) {
+                revert(0, 0)
+            }
+        }
         uint256 wstEthBalancePost = IWStETH(WST_ETH).balanceOf(address(this));
         uint256 wstEthAmount = wstEthBalancePost - wstEthBalancePre;
         return (wstEthAmount);
```