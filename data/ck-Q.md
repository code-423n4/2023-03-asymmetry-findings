## Weight calculation can result in some derivatives failing to get a proportional stake

```solidity
uint256 ethAmount = (msg.value * weight) / totalWeight;
```

If `msg.value` is a low value and a derivative has low weight, there are instances where the `ethAmount` will be zero. 