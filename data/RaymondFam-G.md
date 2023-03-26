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