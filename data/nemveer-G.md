## 1. Checking `ethAmountToRebalance == 0` outside for loop in `rebalanceToWeights()`
No need to check ethAmountToRebalance in each iteration. 

```
< |  SafEth    ·  rebalanceToWeights  ·          -  ·          -  ·        689350  ·            2  ·          -  │
---
> |  SafEth    ·  rebalanceToWeights  ·          -  ·          -  ·        689303  ·            2  ·          -  │
```
```solidity
File: contracts/SafEth/SafEth.sol
+        if(ethAmountToRebalance != 0){
         for (uint i = 0; i < derivativeCount; i++) {
-            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
+            if (weights[i] == 0 ) continue;
             uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                 totalWeight;
             // Price will change due to slippage
             derivatives[i].deposit{value: ethAmount}();
         }
+        }
         emit Rebalanced();
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L138-L155


## 2. Extra division and multiplication can be cancelled out while reducing rounding off errors.
10**18 are multiplied and divided to the same variable which can be ignored. 

```
---
Before
< |  SafEth    ·  stake          ·      45686  ·     660505  ·        353096  ·            4  ·          -  │
---
After
> |  SafEth    ·  stake          ·      45594  ·     660035  ·        352815  ·            4  ·          -  │
```

```solidity
@@ -71,14 +71,13 @@ contract SafEth is
         for (uint i = 0; i < derivativeCount; i++)
             underlyingValue +=
                 (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
-                    derivatives[i].balance()) /
-                10 ** 18;
+                    derivatives[i].balance());
 
         uint256 totalSupply = totalSupply();
         uint256 preDepositPrice; // Price of safETH in regards to ETH
         if (totalSupply == 0)
             preDepositPrice = 10 ** 18; // initializes with a price of 1
-        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
+        else preDepositPrice = (underlyingValue) / totalSupply;
 
         uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
         for (uint i = 0; i < derivativeCount; i++) {
@@ -91,11 +90,11 @@ contract SafEth is
             uint256 depositAmount = derivative.deposit{value: ethAmount}();
             uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
                 depositAmount
-            ) * depositAmount) / 10 ** 18;
+            ) * depositAmount);
             totalStakeValueEth += derivativeReceivedEthValue;
         }
         // mintAmount represents a percentage of the total assets in the system
-        uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
+        uint256 mintAmount = (totalStakeValueEth) / preDepositPrice;
         _mint(msg.sender, mintAmount);
         emit Staked(msg.sender, msg.value, mintAmount);
     }

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63-L101