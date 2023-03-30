In the [SafeEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L138) contract, the loop in the function `rebalanceToWeights` can be modified to save gas by decreasing the number of iterations.

The second part of the [conditional](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L148), `ethAmountToRebalance == 0` can be taken out of the loop. The following can be done: 

```
        if(ethAmountToRebalance == 0) revert();

        for (uint i = 0; i < derivativeCount; i++) {
            if (weights[i] == 0) continue;
            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                totalWeight;
            // Price will change due to slippage
            derivatives[i].deposit{value: ethAmount}();
        }
```

Gas is only saved in the case where `ethAmountToRebalance` equals 0, as the loop would not be executed.