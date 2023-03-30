## 1. Rounding off leads to dust being locked in safEth.sol forever
- In `stake()` function, msg.value is deposited in each derivatives respective to it's weight.
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L88
```
uint256 ethAmount = (msg.value * weight) / totalWeight;

// This is slightly less than ethAmount because slippage
uint256 depositAmount = derivative.deposit{value: ethAmount}();
```
- But, in doing so, due to rounding off in solidity, there's some dust left in contract. Which should ideally be returned to msg.sender
- Over the time, the amount of dust can become significant and there's no way to get it back from contract.

## 2. `totalStakeValueEth` will always be wrong if `poolCanDeposit` == false
- In case of rETH, amount is deposited into rETH pool if `poolCanDeposit == true`. Otherwise it is swapped with rETH via uniswap pool.
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L91-L95
```
uint256 depositAmount = derivative.deposit{value: ethAmount}();
uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
          depositAmount
          ) * depositAmount) / 10 ** 18;
          totalStakeValueEth += derivativeReceivedEthValue;
```
- But as you can see in the snippet, price is fetched after swap happens, which will always be slightly lower, resulting in `totalStakeValueEth` being lesser. 
- Which in turn, mints the lesser safEth to user

## 3. gas cost can be reduced by removing the derivative
- In safEth, owner can inactive a derivative market by changing it's weight to zero.
- But it's never removed from the array.
- So, anytime `stake`, `unstake`, `rebalanceToWeights`, `adjustWeight` or `addDerivative` is called, it uses unnecessary gas which can be prevented by removing the derivative from array

## 4. It's possible that amount is deposited into pool and yet price of swap will be fetched
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L92-L94
**when:** RocketPool's `getMaximumDepositPoolSize()` is reached **right after** user's deposit. 
**Proof of Concept:**
1.  Rocket pool getMaximumDepositPoolSize is at 5000 eth currently. 
2. Assuming that current pool deposit of rocket pool are at 4980 eth. 
3. Now Alice `stake` amount is distributed in such a way that 20 eth is about to deposit() in reth derivative.
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L147-L149
As you can seen the derivative will process the deposit as `poolCanDeposit` function returns true. 
 https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L211-L215
Now to fetch the `totalStakedValueEth` `stake` function calls the `ethPerDerivative` with 20 eth as arguement. But now `poolCanDeposit` will return false and Uniswap v3 pool prices will be fetched, which is incorrect.  
 https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L92-L94
- This may increase or decrease the totalStakedValueEth, depending upon the pool's price, which can influence the users` SafEth proportion. 

## 5. `isStakingPaused` of lido is not checked during deposit in WstEth.sol
https://github.com/lidofinance/lido-dao/blob/master/contracts/0.4.24/Lido.sol#L679
- As you can see, in `_submit` function of Lido, there's a check whether the staking is paused
- In such scenario when staking to Lido is paused, WstEth should use the swap method as done in case of rETH(when poolCanDeposit == false)

## 6. `maxSlippage` should have some upper limit
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202-L208

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L58-L60

- While, changing the slippage of any derivative contract, there is no check on slippage
- Unintended or accidently set high slippage can lead to significant loss of user's funds while swapping