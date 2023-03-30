1. In WstEth.sol, the `ethPerDerivative` is hardcoded to return 1 while both rETH and sfrxETH is using a more native conversion within their corresponding protocol.
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86

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

2. 
`unstake()` in safETH would be blocked if any derivative gets blocked during `withdraw`
The system essentially comes to a halt if any of stETH/frxETH/rETH stops their withdrawal.
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108

```
    function unstake(uint256 _safEthAmount) external {
        require(pauseUnstaking == false, "unstaking is paused");
        uint256 safEthTotalSupply = totalSupply();
        uint256 ethAmountBefore = address(this).balance;

        for (uint256 i = 0; i < derivativeCount; i++) {
            // withdraw a percentage of each asset based on the amount of safETH
            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
            if (derivativeAmount == 0) continue; // if derivative empty ignore
            derivatives[i].withdraw(derivativeAmount);
        }
```

