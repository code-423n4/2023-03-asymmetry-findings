L-01:`maxSlippage` Slippage does not provide real protection
Currently individual `derivatives` are protected against slippage when exchanging, but the estimated prices are all from the `pool price` of the same transaction
When the price drops sharply, the estimated price also drops sharply, and `minOut` also drops sharply, resulting in very limited protection from slippage
It is recommended to calculate `minOut` without adding `pool price` to the calculation, and to calculate `minOut` directly based on the expected eth

```solidity
contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {
    function withdraw(uint256 _amount) external onlyOwner {
...
-        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
-            (10 ** 18 - maxSlippage)) / 10 ** 18;    
+        uint256 minOut = (_amount / 10 ** 18 *
+            (10 ** 18 - maxSlippage)) / 10 ** 18;    

```
```solidity
contract Reth is IDerivative, Initializable, OwnableUpgradeable {
...
    function deposit() external payable onlyOwner returns (uint256) {
-           uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
+           uint256 minOut = (((msg.value / 10 ** 18) *
                ((10 ** 18 - maxSlippage))) / 10 ** 18);    
```    

L-02:stake()/unstake() minAmount/maxAmount Problem
1. take() only determines maxAmount for this time, and does not add how many shares the current user already has
2. unstake() does not limit `_safEthAmount`, so after unstake, the value of the user's remaining shares may be less than minAmount.

It is recommended to add a judgment to the existing shares, but not this minAmount/maxAmount does not work

L-03:adjustWeight() lacks the judgment _derivativeIndex < derivativeCount
If _derivativeIndex > derivativeCount, the adjustment is invalid
Suggestion.
```solidity
    function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {

+       require(_derivativeIndex<derivativeCount,"bad index");
        weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit WeightChange(_derivativeIndex, _weight);
    }
```

L-04:addDerivative() Missing to determine whether `_contractAddress` exists or not
If the derivative address is the same as the old one, it will lead to double calculation of the total assets when `stake()`, if `underlyingValue` is too big, leading to bigger `preDepositPrice` and inaccurate calculation of `mintAmount`.

It is suggested that the loop determine whether the same address already exists in the current list, if existence then revert



L-05:SafEthStorage lack of GAP placeholder, not conducive to subsequent upgrades resulting in storage conflicts