# [G-01] State variable `derivativeCount` should not be looked up in every loop of a for-loop

There are 7 instances of this issue:

```
File: contracts/SafEth/SafEth.sol

 71:       for (uint i = 0; i < derivativeCount; i++)
 84:       for (uint i = 0; i < derivativeCount; i++) {
113:       for (uint i = 0; i < derivativeCount; i++) {
140:       for (uint i = 0; i < derivativeCount; i++) {
147:       for (uint i = 0; i < derivativeCount; i++) {
171:       for (uint i = 0; i < derivativeCount; i++)
191:       for (uint i = 0; i < derivativeCount; i++)
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191


```
diff --git forkSrcPrefix/contracts/SafEth/SafEth.sol forkDstPrefix/contracts/SafEth/SafEth.sol
index ebadb4bbf2b55e7a28b0334c5dd3ea40aa163456..85b27ebb620f7ad1566145677d3b17ad4a2c8517 100644
--- forkSrcPrefix/contracts/SafEth/SafEth.sol
+++ forkDstPrefix/contracts/SafEth/SafEth.sol
@@ -68,7 +68,8 @@ contract SafEth is
         uint256 underlyingValue = 0;
 
         // Getting underlying value in terms of ETH for each derivative
-        for (uint i = 0; i < derivativeCount; i++)
+        uint _derivativeCount = derivativeCount;
+        for (uint i = 0; i < _derivativeCount; i++)
             underlyingValue +=
                 (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                     derivatives[i].balance()) /
@@ -81,7 +82,8 @@ contract SafEth is
         else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
 
         uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
-        for (uint i = 0; i < derivativeCount; i++) {
+        uint _derivativeCount = derivativeCount;
+        for (uint i = 0; i < _derivativeCount; i++) {
             uint256 weight = weights[i];
             IDerivative derivative = derivatives[i];
             if (weight == 0) continue;
@@ -110,7 +112,8 @@ contract SafEth is
         uint256 safEthTotalSupply = totalSupply();
         uint256 ethAmountBefore = address(this).balance;
 
-        for (uint256 i = 0; i < derivativeCount; i++) {
+        uint _derivativeCount = derivativeCount;
+        for (uint256 i = 0; i < _derivativeCount; i++) {
             // withdraw a percentage of each asset based on the amount of safETH
             uint256 derivativeAmount = (derivatives[i].balance() *
                 _safEthAmount) / safEthTotalSupply;
@@ -137,14 +140,16 @@ contract SafEth is
     */
     function rebalanceToWeights() external onlyOwner {
         uint256 ethAmountBefore = address(this).balance;
-        for (uint i = 0; i < derivativeCount; i++) {
+        uint _derivativeCount = derivativeCount;
+        for (uint i = 0; i < _derivativeCount; i++) {
             if (derivatives[i].balance() > 0)
                 derivatives[i].withdraw(derivatives[i].balance());
         }
         uint256 ethAmountAfter = address(this).balance;
         uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
 
-        for (uint i = 0; i < derivativeCount; i++) {
+        uint _derivativeCount = derivativeCount;
+        for (uint i = 0; i < _derivativeCount; i++) {
             if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
             uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                 totalWeight;
@@ -168,7 +173,8 @@ contract SafEth is
     ) external onlyOwner {
         weights[_derivativeIndex] = _weight;
         uint256 localTotalWeight = 0;
-        for (uint256 i = 0; i < derivativeCount; i++)
+        uint _derivativeCount = derivativeCount;
+        for (uint256 i = 0; i < _derivativeCount; i++)
             localTotalWeight += weights[i];
         totalWeight = localTotalWeight;
         emit WeightChange(_derivativeIndex, _weight);
@@ -188,7 +194,8 @@ contract SafEth is
         derivativeCount++;
 
         uint256 localTotalWeight = 0;
-        for (uint256 i = 0; i < derivativeCount; i++)
+        uint _derivativeCount = derivativeCount;
+        for (uint256 i = 0; i < _derivativeCount; i++)
             localTotalWeight += weights[i];
         totalWeight = localTotalWeight;
         emit DerivativeAdded(_contractAddress, _weight, derivativeCount);

```

# [G-02] Access the state variable `derivatives` and their `balance` multiple times in for-loop

There are 3 instances of this issue:

```
File: contracts/SafEth/SafEth.sol

 72:            underlyingValue +=
 73:                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
 74:                    derivatives[i].balance()) /

115:            uint256 derivativeAmount = (derivatives[i].balance() *
116:                _safEthAmount) / safEthTotalSupply;
117:            if (derivativeAmount == 0) continue; // if derivative empty ignore
118:            derivatives[i].withdraw(derivativeAmount); 

141:            if (derivatives[i].balance() > 0)
142:                derivatives[i].withdraw(derivatives[i].balance());
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L72-L74

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L115-L118

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L141-L142

```
diff --git forkSrcPrefix/contracts/SafEth/SafEth.sol forkDstPrefix/contracts/SafEth/SafEth.sol
index ebadb4bbf2b55e7a28b0334c5dd3ea40aa163456..b2c6be428a68ab0b29ff6697002bdf04d882f624 100644
--- forkSrcPrefix/contracts/SafEth/SafEth.sol
+++ forkDstPrefix/contracts/SafEth/SafEth.sol
@@ -69,9 +69,11 @@ contract SafEth is
 
         // Getting underlying value in terms of ETH for each derivative
         for (uint i = 0; i < derivativeCount; i++)
+            IDerivative derivative = derivatives[i];
+            uint256 derivativeBalance = derivative.balance();
             underlyingValue +=
-                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
-                    derivatives[i].balance()) /
+                (derivative.ethPerDerivative(derivativeBalance) *
+                    derivativeBalance) /
                 10 ** 18;
 
         uint256 totalSupply = totalSupply();
@@ -112,10 +114,11 @@ contract SafEth is
 
         for (uint256 i = 0; i < derivativeCount; i++) {
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
@@ -138,8 +141,10 @@ contract SafEth is
     function rebalanceToWeights() external onlyOwner {
         uint256 ethAmountBefore = address(this).balance;
         for (uint i = 0; i < derivativeCount; i++) {
-            if (derivatives[i].balance() > 0)
-                derivatives[i].withdraw(derivatives[i].balance());
+            IDerivative derivative = derivatives[i];
+            uint256 derivativeBalance = derivative.balance();
+            if (derivativeBalance > 0)
+                derivative.withdraw(derivativeBalance);
         }
         uint256 ethAmountAfter = address(this).balance;
         uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;

```


# [G-03] Access the state variable `weights` multiple times in for-loop

```
File: contracts/SafEth/SafEth.sol

148:            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
149:            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L148-L149

```
diff --git forkSrcPrefix/contracts/SafEth/SafEth.sol forkDstPrefix/contracts/SafEth/SafEth.sol
index ebadb4bbf2b55e7a28b0334c5dd3ea40aa163456..02f1a37c9b5b55918fa60904b071460371195278 100644
--- forkSrcPrefix/contracts/SafEth/SafEth.sol
+++ forkDstPrefix/contracts/SafEth/SafEth.sol
@@ -145,8 +145,9 @@ contract SafEth is
         uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
 
         for (uint i = 0; i < derivativeCount; i++) {
-            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
-            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
+            uint256 weight = weights[i];
+            if (weight == 0 || ethAmountToRebalance == 0) continue;
+            uint256 ethAmount = (ethAmountToRebalance * weight) /
                 totalWeight;
             // Price will change due to slippage
             derivatives[i].deposit{value: ethAmount}();

```