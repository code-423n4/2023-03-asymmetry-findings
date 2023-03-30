In WstEth.sol, the `ethPerDerivative` is hardcoded to return 1 while the sfrxETH is using an oracle.
Actually price of stETH can be fetched from the curveAMM too using the get_dy.

Updated Approach

```
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        return 10 ** 18 * (IStEthEthPool(LIDO_CRV_POOL).get_dy(1, 0, stEthBal) / stEthBal);
    }
```

Current Approach
```    
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
    }
```