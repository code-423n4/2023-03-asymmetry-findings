* Function `ethPerDerivative` in WstETH.sol, should be
```
return IWStETH(WST_ETH).stEthPerToken();
```
instead of
 ```
return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L87