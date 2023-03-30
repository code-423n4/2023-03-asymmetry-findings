In WstEth.sol, the input `_amount` is unnecessary since it's not used.

```    
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
    }
```