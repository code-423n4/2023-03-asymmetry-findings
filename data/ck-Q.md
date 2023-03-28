## Validate minimum and maximum values when setting slippage

```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

Currently there are no restrictions on the amount of slippage that can be set. To prevent unintended effects, the input should be validated for acceptable minimum and maximum slippage.

## Weight calculation can result in some derivatives failing to get a proportional stake

```solidity
uint256 ethAmount = (msg.value * weight) / totalWeight;
```

If `msg.value` is a low value and a derivative has low weight, there are instances where the `ethAmount` will be zero. 