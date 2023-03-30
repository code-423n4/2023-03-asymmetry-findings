### Table of Contents
- [FINDINGS](#findings)
- [Cache storage values in memory to minimize SLOADs](#cache-storage-values-in-memory-to-minimize-sloads)
  - [SafEth.sol.stake(): derivativeCount being a storage variable should not be looped into as it's too costly,consider caching it](#safethsolstake-derivativecount-being-a-storage-variable-should-not-be-looped-into-as-its-too-costlyconsider-caching-it)
  - [SafEth.sol.unstake(): derivativeCount should be cached especially because it's being used in a for loop](#safethsolunstake-derivativecount-should-be-cached-especially-because-its-being-used-in-a-for-loop)
  - [SafEth.sol.rebalanceToWeights(): derivativeCount should be cached](#safethsolrebalancetoweights-derivativecount-should-be-cached)
  - [SafEth.sol.addDerivative(): derivativeCount should be cached](#safethsoladdderivative-derivativecount-should-be-cached)
- [Emitting storage values instead of the memory one.](#emitting-storage-values-instead-of-the-memory-one)
- [Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead](#usage-of-uintsints-smaller-than-32-bytes-256-bits-incurs-overhead)
- [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)
- [`keccak256()` should only need to be called on a specific string literal once](#keccak256-should-only-need-to-be-called-on-a-specific-string-literal-once)
- [Functions guaranteed to revert when called by normal users can be marked `payable`](#functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable)

## FINDINGS
NB: Some functions have been truncated where necessary to just show affected parts of the code
Through out the report some places might be denoted with audit tags to show the actual place affected.

## Cache storage values in memory to minimize SLOADs
The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L63-L101
### SafEth.sol.stake(): derivativeCount being a storage variable should not be looped into as it's too costly,consider caching it
```solidity
File: /contracts/SafEth/SafEth.sol
63:    function stake() external payable {

70:        // Getting underlying value in terms of ETH for each derivative
71:        for (uint i = 0; i < derivativeCount; i++)

84:        for (uint i = 0; i < derivativeCount; i++) {
```

```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..48d995a 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -68,7 +68,8 @@ contract SafEth is
         uint256 underlyingValue = 0;

         // Getting underlying value in terms of ETH for each derivative
-        for (uint i = 0; i < derivativeCount; i++)
+        uint256 _derivativeCount = derivativeCount;
+        for (uint i = 0; i < _derivativeCount; i++)
             underlyingValue +=
                 (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                     derivatives[i].balance()) /
@@ -81,7 +82,7 @@ contract SafEth is
         else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

         uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
-        for (uint i = 0; i < derivativeCount; i++) {
+        for (uint i = 0; i < _derivativeCount; i++) {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L108-L129
### SafEth.sol.unstake(): derivativeCount should be cached especially because it's being used in a for loop
```solidity
File: /contracts/SafEth/SafEth.sol
108:    function unstake(uint256 _safEthAmount) external {

113:        for (uint256 i = 0; i < derivativeCount; i++) {
114:            // withdraw a percentage of each asset based on the amount of safETH
115:            uint256 derivativeAmount = (derivatives[i].balance() *
116:                _safEthAmount) / safEthTotalSupply;
117:            if (derivativeAmount == 0) continue; // if derivative empty ignore
118:            derivatives[i].withdraw(derivativeAmount);
```

```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..3398ec0 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -109,8 +109,9 @@ contract SafEth is
         require(pauseUnstaking == false, "unstaking is paused");
         uint256 safEthTotalSupply = totalSupply();
         uint256 ethAmountBefore = address(this).balance;
+        uint256 _derivativeCount = derivativeCount;

-        for (uint256 i = 0; i < derivativeCount; i++) {
+        for (uint256 i = 0; i < _derivativeCount; i++) {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L138-L155
### SafEth.sol.rebalanceToWeights(): derivativeCount should be cached
```solidity
File: /contracts/SafEth/SafEth.sol
138:    function rebalanceToWeights() external onlyOwner {
139:        uint256 ethAmountBefore = address(this).balance;
140:        for (uint i = 0; i < derivativeCount; i++) {
141:            if (derivatives[i].balance() > 0)
142:                derivatives[i].withdraw(derivatives[i].balance());
143:        }
144:        uint256 ethAmountAfter = address(this).balance;
145:        uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;

147:        for (uint i = 0; i < derivativeCount; i++) {
148:            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
149:            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
150:                totalWeight;
151:            // Price will change due to slippage
152:            derivatives[i].deposit{value: ethAmount}();
153:       }
154:        emit Rebalanced();
155:    }
```

```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..d7cf84a 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -137,14 +137,15 @@ contract SafEth is
     */
     function rebalanceToWeights() external onlyOwner {
         uint256 ethAmountBefore = address(this).balance;
-        for (uint i = 0; i < derivativeCount; i++) {
+        uint256 _derivativeCount = derivativeCount;
+        for (uint i = 0; i < _derivativeCount; i++) {
             if (derivatives[i].balance() > 0)
                 derivatives[i].withdraw(derivatives[i].balance());
         }
         uint256 ethAmountAfter = address(this).balance;
         uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;

-        for (uint i = 0; i < derivativeCount; i++) {
+        for (uint i = 0; i < _derivativeCount; i++) {
             if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
             uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                 totalWeight;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182-L195
### SafEth.sol.addDerivative(): derivativeCount should be cached
```solidity
File: /contracts/SafEth/SafEth.sol
182:    function addDerivative(
183:        address _contractAddress,
184:        uint256 _weight
185:    ) external onlyOwner {
186:        derivatives[derivativeCount] = IDerivative(_contractAddress);
187:        weights[derivativeCount] = _weight;
188:        derivativeCount++;

190:        uint256 localTotalWeight = 0;
191:        for (uint256 i = 0; i < derivativeCount; i++)
192:            localTotalWeight += weights[i];
193:        totalWeight = localTotalWeight;
194:        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
195:    }
```

## Emitting storage values instead of the memory one.
Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead: This would save us ~100 gas as we avoid one SLOAD

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L214-L217
```solidity
File: /contracts/SafEth/SafEth.sol
214:    function setMinAmount(uint256 _minAmount) external onlyOwner {
215:        minAmount = _minAmount;
216:        emit ChangeMinAmount(minAmount);
217:    }
```

```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..17ea481 100644
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

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L223-L226
```solidity
File: /contracts/SafEth/SafEth.sol
223:    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
224:        maxAmount = _maxAmount;
225:        emit ChangeMaxAmount(maxAmount);
226:    }
```

```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..39482eb 100644
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

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L232-L235
```solidity
File: /contracts/SafEth/SafEth.sol
232:    function setPauseStaking(bool _pause) external onlyOwner {
233:        pauseStaking = _pause;
234:        emit StakingPaused(pauseStaking);
235:    }
```

```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..a9c4659 100644
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

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L241-L244
```solidity
File: /contracts/SafEth/SafEth.sol
241:    function setPauseUnstaking(bool _pause) external onlyOwner {
242:        pauseUnstaking = _pause;
243:        emit UnstakingPaused(pauseUnstaking);
244:    }
```

```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..a0eb13f 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -240,7 +240,7 @@ contract SafEth is
     */
     function setPauseUnstaking(bool _pause) external onlyOwner {
         pauseUnstaking = _pause;
-        emit UnstakingPaused(pauseUnstaking);
+        emit UnstakingPaused(_pause);
     }
```


## Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Use a larger size then downcast where needed

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L83-L89
```solidity
File: /contracts/SafEth/derivatives/Reth.sol

//@audit: uint24 _poolFee
83:    function swapExactInputSingleHop(
84:        address _tokenIn,
85:        address _tokenOut,
86:        uint24 _poolFee,
87:        uint256 _amountIn,
88:        uint256 _minOut
89:    ) private returns (uint256 amountOut) {
```

## Using unchecked blocks to save gas
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
[see resource](https://github.com/ethereum/solidity/issues/10695)

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L201
```solidity
File: /contracts/SafEth/derivatives/Reth.sol
201:            uint256 rethMinted = rethBalance2 - rethBalance1;
```
The operation `rethBalance2 - rethBalance1` cannot underflow due to the require statement on [Line 200](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L200) that ensures that `rethBalance2` is greater than `rethBalance1` before performing the operation

```diff
diff --git a/contracts/SafEth/derivatives/Reth.sol b/contracts/SafEth/derivatives/Reth.sol
index b6e0694..3a0f754 100644
--- a/contracts/SafEth/derivatives/Reth.sol
+++ b/contracts/SafEth/derivatives/Reth.sol
@@ -198,7 +198,10 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
             rocketDepositPool.deposit{value: msg.value}();
             uint256 rethBalance2 = rocketTokenRETH.balanceOf(address(this));
             require(rethBalance2 > rethBalance1, "No rETH was minted");
-            uint256 rethMinted = rethBalance2 - rethBalance1;
+            uint256 rethMinted;
+            unchecked{
+                 rethBalance2 - rethBalance1;
+            }
             return (rethMinted);
         }
     }
```


## `keccak256()` should only need to be called on a specific string literal once

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to `bytes4` should also only be done once

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L69-L71
```solidity
File: /contracts/SafEth/derivatives/Reth.sol
69:                keccak256(
70:                    abi.encodePacked("contract.address", "rocketTokenRETH")
71:                )
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L124-L126
```solidity
File: /contracts/SafEth/derivatives/Reth.sol
124:                keccak256(
125:                    abi.encodePacked("contract.address", "rocketDepositPool")
126:                )
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L135-L140
```solidity
File: /contracts/SafEth/derivatives/Reth.sol
135:                keccak256(
136:                    abi.encodePacked(
137:                        "contract.address",
138:                        "rocketDAOProtocolSettingsDeposit"
139:                    )
140:                )
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L161-L163
```solidity
File: /contracts/SafEth/derivatives/Reth.sol
161:                keccak256(
162:                    abi.encodePacked("contract.address", "rocketDepositPool")
163:                )
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L190-L192
```solidity
File: /contracts/SafEth/derivatives/Reth.sol
190:                    keccak256(
191:                        abi.encodePacked("contract.address", "rocketTokenRETH")
192:                    )
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L232-L234
```solidity
File: /contracts/SafEth/derivatives/Reth.sol
232:                keccak256(
233:                    abi.encodePacked("contract.address", "rocketTokenRETH")
234:                )
```


## Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.The extra opcodes avoided costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48
```solidity
File: /contracts/SafEth/derivatives/WstEth.sol
48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

56:    function withdraw(uint256 _amount) external onlyOwner {

73:    function deposit() external payable onlyOwner returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51
```solidity
File: /contracts/SafEth/derivatives/SfrxEth.sol

51:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

60:    function withdraw(uint256 _amount) external onlyOwner {

94:    function deposit() external payable onlyOwner returns (uint256) {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L138
```solidity
File: /contracts/SafEth/SafEth.sol

138:    function rebalanceToWeights() external onlyOwner {

165:    function adjustWeight(
166:        uint256 _derivativeIndex,
167:        uint256 _weight
168:    ) external onlyOwner {

182:    function addDerivative(
183:        address _contractAddress,
184:        uint256 _weight
185:    ) external onlyOwner {

202:    function setMaxSlippage(
203:        uint _derivativeIndex,
204:        uint _slippage
205:    ) external onlyOwner {

214:    function setMinAmount(uint256 _minAmount) external onlyOwner {

223:    function setMaxAmount(uint256 _maxAmount) external onlyOwner {

232:    function setPauseStaking(bool _pause) external onlyOwner {

241:    function setPauseUnstaking(bool _pause) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L58
```solidity
File: /contracts/SafEth/derivatives/Reth.sol

58:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

107:    function withdraw(uint256 amount) external onlyOwner {

156:    function deposit() external payable onlyOwner returns (uint256) {
```
