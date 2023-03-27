# Asymmetry Low Risk and Non-Critical Issues
## Summary
| Risk      | Title | File | Instances
| ----------- | ----------- | ----------- | ----------- |
| L-01      | Use fixed compiler version | - | 4 |
| L-02      | Wrong index emitted in `DerivativeAdded` event | SafEth.sol | 1 |
| L-03      | OwnableUpgradeable: Does not implement 2-Step-Process for transferring ownership | SafEth.sol | 1 |
| L-04      | `SafEth.stake`: check `ethAmount` for zero instead of `weight` | SafEth.sol | 1 |
| L-05      | Fixed slippage setting is inefficient because traded amounts can vary | - | 3 |
| N-01      | Remove unnecessary imports | - | 5 |  
| N-02      | `SafEth` contract `receive` function can be dangerous for users | SafEth.sol | 1 |  
| N-03      | `SafEth`: Use `address(this).balance` instead of calculating difference | SafEth.sol | 2 |  
| N-04      | `Reth.ethPerDerivative` calculation can be simplified | Reth.sol | 1 |  


## [L-01] Use fixed compiler version
All in scope contracts use `^0.8.13` as compiler version.  
They should use a fixed version, i.e. `0.8.13`, to make sure the contracts are always compiled with
the intended version.  

## [L-02] Wrong index emitted in `DerivativeAdded` event
The `DerivativeAdded` event is defined as this:  
[Link](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L29-L33)  
```solidity
event DerivativeAdded(
    address indexed contractAddress,
    uint weight,
    uint index
);
```

So the third parameter is the index of the new derivative in the `derivatives` mapping.

The `addDerivative` function that emits this event does it like this:  
[Link](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182-L195)  
```solidity
function addDerivative(
    address _contractAddress,
    uint256 _weight
) external onlyOwner {
    derivatives[derivativeCount] = IDerivative(_contractAddress);
    weights[derivativeCount] = _weight;
    // @audit increment happens
    derivativeCount++;


    uint256 localTotalWeight = 0;
    for (uint256 i = 0; i < derivativeCount; i++)
        localTotalWeight += weights[i];
    totalWeight = localTotalWeight;
    // @audit should be derivativeCount - 1
    emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
}
```

The issue is that `derivativeCount` is not the correct index.  
The correct index is `derivativeCount - 1` since `derivativeCount` is incremented in the function.  

The impact of this is that any off-chain components that process this event will misbehave.  

This can possibly result in the `adjustWeight` function being called with the wrong index.  

## [L-03] OwnableUpgradeable: Does not implement 2-Step-Process for transferring ownership
`SafEth` inherits from openzeppelin's `OwnableUpgradeable` contract.  
This contract does not implement a `2-Step-Process` for transferring ownership.  
So ownership of the contract can easily be lost when making a mistake when transferring ownership.  

Consider using the `OwnableUpgradeable2Step` contract ([https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol)) instead.  

Note that I did not flag this as an issue for the derivative contracts. The derivative contracts are only ever owned by a single `SafEth` contract and ownership will not be transferred. So it is ok to use the regular `OwnableUpgradeable` contract for the derivatives.  

## [L-04] `SafEth.stake`: check `ethAmount` for zero instead of `weight`
[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L84-L96](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L84-L96)  

The `SafEth.stake` function skips a derivative if `weight == 0`.  

However this can introduce an issue if `weight` is really small.  

`weight` can have a very small non-zero value which causes `ethAmount` to be zero:  
`uint256 ethAmount = (msg.value * weight) / totalWeight;`

E.g. `0.5 ETH * 1 / 1e18 = 0`.  

Some derivatives will revert when trying to deposit a zero amount.  

Therefore it is safer to check `ethAmount` for zero instead of `weight`.  

```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..56afefb 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -84,9 +84,8 @@ contract SafEth is
         for (uint i = 0; i < derivativeCount; i++) {
             uint256 weight = weights[i];
             IDerivative derivative = derivatives[i];
-            if (weight == 0) continue;
             uint256 ethAmount = (msg.value * weight) / totalWeight;
-
+            if (ethAmount == 0) continue;
             // This is slightly less than ethAmount because slippage
             uint256 depositAmount = derivative.deposit{value: ethAmount}();
             uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
```

## [L-05] Fixed slippage setting is inefficient because traded amounts can vary
[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L58-L60](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L58-L60)  

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48-L50](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48-L50)  

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53)  

All three derivative contracts (`Reth.sol`, `SfrxEth.sol` and `WstEth.sol`) use a fixed slippage percentage. This means that while the `owner` can change the slippage for each derivative contract, the slippage setting for a given contract is the same whether a small amount of funds is traded or a big amount of funds is traded.  

This means that the slippage percentage must be chosen big enough such that the maximum amount of funds possible can be traded (after talking to the sponsor this is in the range of 100 to 1000 ETH).  

(The only functionality that trades more funds is rebalancing. But the `owner` can pause staking, increase the slippage percentage, rebalance, decrease the slippage percentage and unpause staking again. So there is no need to set the slippage percentage so high to allow rebalancing all of the time.)

Obviously this slippage percentage is not the best percentage when a small amount is traded.  

I estimate this to be of "Low" severity because currently the liquidity in all pools that the protocol integrates with is sufficient such that the price moves only a little even with the maximum amount of funds that will be traded.  

However the liquidity may dry out and then a single slippage percentage offers no slippage protection when small amounts are traded.  

Therefore it might be beneficial to let users choose the slippage percentage for themselves.  

## [N-01] Remove unnecessary imports
Unnecessary imports should be removed from the source files to improve readability.

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L5](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L5)  

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L6](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L6)  

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L7](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L7)  

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L8](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L8)  

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L5](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L5)  

## [N-02] `SafEth` contract `receive` function can be dangerous for users
The `SafEth` contract implements a `receive` function:  

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L246](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L246)  

This function is needed for the derivative contracts to send ETH which is then sent along further to a user that wants to withdraw.  

The issue is that it's easy for a user to send the ETH directly to the `SafEth` contract instead of using the `SafEth.stake` function. This would cause a complete loss of funds.  

This is a mistake by the user but it might still be beneficial to check within the `receive` function that `msg.sender` is one of the derivative contracts.  

Thereby a user can only send ETH to the `SafEth` contract by calling the `SafEth.stake` function which is the correct function to call.  

## [N-03] `SafEth`: Use `address(this).balance` instead of calculating difference
[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L139-L145](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L139-L145)  

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L111-L122](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L111-L122)  

The `SafEth` contract does not hold any ETH across transactions. So it does not need to calculate `ethAmountAfter - ethAmountBefore`.  

It can just use `address(this).balance`.  

This simplifies the Code, makes it more readable and also saves a bit of Gas.  

Fix:  
```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..5a2286f 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -108,7 +108,6 @@ contract SafEth is
     function unstake(uint256 _safEthAmount) external {
         require(pauseUnstaking == false, "unstaking is paused");
         uint256 safEthTotalSupply = totalSupply();
-        uint256 ethAmountBefore = address(this).balance;
 
         for (uint256 i = 0; i < derivativeCount; i++) {
             // withdraw a percentage of each asset based on the amount of safETH
@@ -118,8 +117,7 @@ contract SafEth is
             derivatives[i].withdraw(derivativeAmount);
         }
         _burn(msg.sender, _safEthAmount);
-        uint256 ethAmountAfter = address(this).balance;
-        uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;
+        uint256 ethAmountToWithdraw = address(this).balance;
         // solhint-disable-next-line
         (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
             ""
@@ -136,13 +134,11 @@ contract SafEth is
         @dev - Probably not going to be used often, if at all
     */
     function rebalanceToWeights() external onlyOwner {
-        uint256 ethAmountBefore = address(this).balance;
         for (uint i = 0; i < derivativeCount; i++) {
             if (derivatives[i].balance() > 0)
                 derivatives[i].withdraw(derivatives[i].balance());
         }
-        uint256 ethAmountAfter = address(this).balance;
-        uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
+        uint256 ethAmountToRebalance = address(this).balance;
 
         for (uint i = 0; i < derivativeCount; i++) {
             if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
```

## [N-04] `Reth.ethPerDerivative` calculation can be simplified
[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L215](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L215)  

The line:  
```solidity
else return (poolPrice() * 10 ** 18) / (10 ** 18);
```

can be simplified to:
```solidity
else return poolPrice();
```


