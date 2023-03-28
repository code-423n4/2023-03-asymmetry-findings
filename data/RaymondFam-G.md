## For loop pre-condition
In `rebalanceToWeights()` of SatEth.sol, `ethAmountToRebalance == 0` should be moved out of the if block before the for loop to save gas on iterations. This is because if `ethAmountToRebalance` is ever true, it means there is no ETH available to deposit into any of the derivatives.

Consider refactoring the affected code as follows:

[File: SafEth.sol#L147-L155](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147-L155)  

```diff
+        if (ethAmountToRebalance == 0) return; 

        for (uint i = 0; i < derivativeCount; i++) {
-            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
+            if (weights[i] == 0) continue;
            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                totalWeight;
            // Price will change due to slippage
            derivatives[i].deposit{value: ethAmount}();
        }
        emit Rebalanced();
    }
```
## Use array of mappings
When a derivative has turned obsolete, i.e. weights assigned zero and the associated underlyingValue has been fully drained after [`rebalanceToWeights()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138-L155) is called, it does not make sense the derivative will have to be called and skipped each time `unstake()` is called, which incurs additional gas.

Consider adding `derivatives[x]` and `weights[x]` to their respective arrays in [`addDerivative()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-L195) and have the obsolete mappings removed from the arrays via `rebalanceToWeights()` or a separately implemented function when need be and deemed fit. Use the array length to iterate the for loop in [`stake()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71-L96) and [`unstake()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113-L119) instead of `derivativeCount`.

## Avoid comparing boolean expressions to boolean literals
Comparing a boolean value to a boolean literal incurs the `ISZERO` operation and costs more gas than using a boolean expression.

Here are some of the affected code lines that may be refactored as follows:

[File: SafEth.sol#L64](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L64)

```diff
-        require(pauseStaking == false, "staking is paused");
+        require(!pauseStaking, "staking is paused");
```
[File: SafEth.sol#L109](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L109)

```diff
-        require(pauseUnstaking == false, "unstaking is paused");
+        require(!pauseUnstaking, "unstaking is paused");
```
## Non-strict inequalities are cheaper than strict ones
In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).

As an example, consider correspondingly replacing `>=` and `<=` with the strict counterpart `>` and `<` in the following inequality instances:

[File: SafEth.sol#L65-L66](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L65-L66)

```diff
-        require(msg.value >= minAmount, "amount too low");
// Rationale for subtracting 1 on the right side of the inequality:
// x >= 10; [10, 11, 12, ...]
// x > 10 - 1 is the same as x > 9; [10, 11, 12 ...]
+        require(msg.value > minAmount - 1, "amount too low");
-        require(msg.value <= maxAmount, "amount too high");
// Rationale for adding 1 on the right side of the inequality:
// x <= 10; [10, 9, 8, ...]
// x < 10 + 1 is the same as x < 11; [10, 9, 8 ...]
+        require(msg.value < maxAmount + 1, "amount too high");
```
## += and -= cost more gas
`+=` and `-=` generally cost 22 more gas than writing out the assigned equation explicitly. The amount of gas wasted can be quite sizable when repeatedly operated in a loop.

For example, the `+=` instance below may be refactored as follows:

[File: SafEth.sol#L95](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L95)

```diff
-            totalStakeValueEth += derivativeReceivedEthValue;
+            totalStakeValueEth = totalStakeValueEth + derivativeReceivedEthValue;
```
## State variables repeatedly read should be cached
SLOADs cost 100 gas each after the 1st one whereas MLOADs/MSTOREs only incur 3 gas each. As such, storage values read multiple times should be cached in the stack memory the first time (costing only 1 SLOAD) and then re-read from this cache to avoid multiple SLOADs.

For instance, `derivativeCount` in the for loop of `stake()` in SafEth.sol may be cached as follows:

[File: SafEth.sol#L84](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84)

```diff
+        uint256 _derivativeCount = derivativeCount;
-        for (uint i = 0; i < derivativeCount; i++) {
+        for (uint i = 0; i < _derivativeCount; i++) {
```
## Code repeatedly used should be grouped into a modifier
Grouping similar/identical code block into a modifier makes the code base more structured while reducing the contract size.

Here is a specific instance entailed where only a `pauser()` in lieu of [`setPauseStaking()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232-L235) and [`setPauseUnstaking()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241-L244) is needed:

[File: SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)

```solidity
64:        require(pauseStaking == false, "staking is paused");

109:        require(pauseUnstaking == false, "unstaking is paused");
```
## Unused imports
Consider removing unused imports to help save gas on contract deployment.

Here are some of the instances entailed: 

[File: SafEth.sol#L4-L11](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L4-L11)

```diff
- import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
- import "../interfaces/IWETH.sol";
- import "../interfaces/uniswap/ISwapRouter.sol";
- import "../interfaces/lido/IWStETH.sol";
- import "../interfaces/lido/IstETH.sol";
 import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
 import "./SafEthStorage.sol";
 import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
```
## Use of named returns for local variables saves gas
You can have further advantages in term of gas cost by simply using named return values as temporary local variable.

For instance, the code block below may be refactored as follows:

[File: WstEth.sol#L73-L81](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L73-L81)

```diff
-    function deposit() external payable onlyOwner returns (uint256 wstEthAmount) {
+    function deposit() external payable onlyOwner returns (uint256) {
        uint256 wstEthBalancePre = IWStETH(WST_ETH).balanceOf(address(this));
        // solhint-disable-next-line
        (bool sent, ) = WST_ETH.call{value: msg.value}("");
        require(sent, "Failed to send Ether");
        uint256 wstEthBalancePost = IWStETH(WST_ETH).balanceOf(address(this));
-        uint256 wstEthAmount = wstEthBalancePost - wstEthBalancePre;
+        wstEthAmount = wstEthBalancePost - wstEthBalancePre;
-        return (wstEthAmount);
    }
```
## `||` costs less gas than its equivalent `&&`
Rule of thumb: `(x && y)` is `(!(!x || !y))`

Even with the 10k Optimizer enabled: `||`, OR conditions cost less than their equivalent `&&`, AND conditions.

As an example, the `&&` instance below may be refactored as follows:

Note: Comment on the changes made for better readability where deemed fit.

[File: Reth.sol#L147-L149](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L147-L149)

```diff
-            rocketDepositPool.getBalance() + _amount <=
-            rocketDAOProtocolSettingsDeposit.getMaximumDepositPoolSize() &&
-            _amount >= rocketDAOProtocolSettingsDeposit.getMinimumDeposit();

+            (!(rocketDepositPool.getBalance() + _amount >
+            rocketDAOProtocolSettingsDeposit.getMaximumDepositPoolSize() ||
+            _amount < rocketDAOProtocolSettingsDeposit.getMinimumDeposit()));
```
## Function order affects gas consumption
The order of function will also have an impact on gas consumption. Because in smart contracts, there is a difference in the order of the functions. Each position will have an extra 22 gas. The order is dependent on method ID. So, if you rename the frequently accessed function to more early method ID, you can save gas cost. Please visit the following site for further information:

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

## Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.13",
settings: {
  optimizer: {
    enabled: true,
    runs: 1000,
  },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.
