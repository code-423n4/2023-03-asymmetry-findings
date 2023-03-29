1.When there is no limit to the range when setting the maxSlippage, if the slippage set exceeds a reasonable range, it may cause losses to the investor's funds
Recommended setting range：maxSlippage  %0.5 ~ %2

2.When the weights are set improperly, it may cause losses to investors. Improper weightings can cause the value of the derivative to deviate from the underlying asset, which may result in a lack of liquidity or inaccurate pricing for investors seeking to enter or exit positions. This may lead to losses for investors who have not properly assessed the risks and have invested in the derivative. It is important to properly set the weights and perform rigorous risk management to avoid such negative consequences.

3.When unstaking, it is necessary to check whether the user has a sufficient safEth balance.
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108#L129

```solidity
+ require(balanceOf(msg.sender)>=_safEthAmount, "insufficient balance");
```