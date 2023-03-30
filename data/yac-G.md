## 1. Cache `derivatives[i].balance()` in `rebalanceToWeights()`

The `derivatives[i].balance()` external function [is called twice](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L141) in `rebalanceToWeights()` when the `if` statement is entered. Caching the value locally saves gas. Hardhat gas estimates show this caching saves 8154 gas for `rebalanceToWeights()`.

```diff
+ uint256 derivativeBalance = derivatives[i].balance();
- if (derivatives[i].balance() > 0)
+ if (derivativeBalance > 0)
- 	derivatives[i].withdraw(derivatives[i].balance());
+ 	derivatives[i].withdraw(derivativeBalance);
```

## 2. Cache `derivatives[i]` in `stake()`

Caching `derivatives[i]` to a local variable in `stake()` [in this for loop](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71-L75) can save 537 gas.

```diff
- for (uint i = 0; i < derivativeCount; i++)
+ for (uint i = 0; i < derivativeCount; i++) {
+	IDerivative derivative = derivatives[i];
	underlyingValue +=
-		(derivatives[i].ethPerDerivative(derivatives[i].balance()) *
+		(derivative.ethPerDerivative(derivative.balance()) *
-			derivatives[i].balance()) /
+			derivative.balance()) /
		10 ** 18;
+ }
```

## 3. Cache `derivatives[i].balance()` in `stake()`

Caching `derivatives[i].balance()` to a local variable in `stake()` [in this for loop](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71-L75) can save 8145 gas. When this for loop is modified so `derivatives[i]` and `derivatives[i].balance()` are cached, 8592 gas can be saved.

```diff
- for (uint i = 0; i < derivativeCount; i++)
+ for (uint i = 0; i < derivativeCount; i++) {
+	uint256 derivativeBalance = derivatives[i].balance();
	underlyingValue +=
-		(derivatives[i].ethPerDerivative(derivatives[i].balance()) *
+		(derivatives[i].ethPerDerivative(derivativeBalance) *
-			derivatives[i].balance()) /
+			derivativeBalance) /
		10 ** 18;
+ }
```

## 4. Avoid duplicate checks of a constant value

`ethAmountToRebalance` in `rebalanceToWeights()` is not modified in the `for` statement so [checking if this value is zero](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L148) should be moved outside of the for loop to avoid duplicate checks of the same value. This reduces the contract size by 2 bytes and hardhat gas estimates show this saving 55 gas in the maximum gas scenario.

```diff
+ if (ethAmountToRebalance != 0) {
	for (uint i = 0; i < derivativeCount; i++) {
-		if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
+		if (weights[i] == 0) continue;
```

## 5. Move logic `Reth.deposit()` logic inside for loop

The variables `rocketDepositPoolAddress` and `rocketDepositPool` are only used in the `else` block of `Reth.deposit()`. Calculating these values is unnecessary in the `if` block, and therefore wastes unnecessary gas. Move these lines from outside the `if`/`else` block into the `else` block.

```solidity
		// Per RocketPool Docs query addresses each time it is used
        address rocketDepositPoolAddress = RocketStorageInterface(
            ROCKET_STORAGE_ADDRESS
        ).getAddress(
                keccak256(
                    abi.encodePacked("contract.address", "rocketDepositPool")
                )
            );

        RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(
                rocketDepositPoolAddress
            );
```
