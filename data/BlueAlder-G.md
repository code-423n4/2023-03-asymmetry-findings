## [G-01] Unnecessary recalculation of total weights in `addDerivative`

In the `addDerivative` function, the total weight of all the
derivatives is recalculated by iterating through the `weights` storage variable
and individually adding these all up in a for loop.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L191


This is not required since `addDerivative` is only adding 1 weight, and not
adjusting the existing ones. So you can simply add the new weight to the
existing totalWeight with:

```sol
totalWeight = totalWeight + _weight;
```

Gas saving is 522 gas and increases with the number of derivatives.

## [G-02] Unnecessary recalculation of total weights in `adjustWeight`

Similar to above, the `adjustWeight` function is only changing 1 weight and so
we can alter the `totalWeight` variable directly before altering the `weights`
mapping to get the calculation.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L165

```sol
totalWeight = totalWeight - weights[_derivativeIndex] + _weight;
weights[_derivativeIndex] = _weight;
```

Gas saving is 154 and increases with the number of derivatives.

# [G-03] **Loop variable in for loop can be unchecked**

```markdown
## [G-03] Loop variable in for loop can be unchecked 

Overflowing the loop variable in may for loops is very unlikely when iterating
derivatives so we can remove the overflow check to save some gas. This is
because most loops start at 0 and increment by 1 and will never reach the max
uint256 or will run out of gas by then.

This will save about 50 gas per iteration.

For example in the `unstake()` function:

```sol
for (uint256 i = 0; i < derivativeCount; i++) {
    // withdraw a percentage of each asset based on the amount of safETH
    uint256 derivativeAmount = (derivatives[i].balance() * _safEthAmount) / safEthTotalSupply;
    if (derivativeAmount == 0) continue; // if derivative empty ignore
    derivatives[i].withdraw(derivativeAmount);
}
```

Should be changed too

```
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index 849d456..0db8b3b 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -98,11 +98,14 @@ contract SafEth is Initializable, ERC20Upgradeable, OwnableUpgradeable, SafEthSt
         uint256 safEthTotalSupply = totalSupply();
         uint256 ethAmountBefore = address(this).balance;
 
-        for (uint256 i = 0; i < derivativeCount; i++) {
+        for (uint256 i = 0; i < derivativeCount;) {
             // withdraw a percentage of each asset based on the amount of safETH
             uint256 derivativeAmount = (derivatives[i].balance() * _safEthAmount) / safEthTotalSupply;
             if (derivativeAmount == 0) continue; // if derivative empty ignore
             derivatives[i].withdraw(derivativeAmount);
+            unchecked {
+                ++i;
+            }
         }
```

Other places to update:

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L84

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L147

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L171

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L191
```
