## Impact
Slippage is the difference between a trade's expected or requested price and the price at which the trade is effectively executed. It typically occurs in markets experiencing high volatility or low liquidity.

In this protocol, users cannot control the splippage when staking or unstaking derivatives. This value is set and controlled by the owner of the contracts (which can allow another issues).

## Proof of Concept
when users unstake their safETH, the `unstake()` function calls `withdraw()` on each derivative.
```
function unstake(uint256 _safEthAmount) external {
[...]
        for (uint256 i = 0; i < derivativeCount; i++) {
            // withdraw a percentage of each asset based on the amount of safETH
            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
            if (derivativeAmount == 0) continue; // if derivative empty ignore
            derivatives[i].withdraw(derivativeAmount);
[...]
```
This `withdraw()` functions apply some `maxSlippage` when exchanging the derivative to ETH.
```
function withdraw(uint256 _amount) external onlyOwner {
        IWStETH(WST_ETH).unwrap(_amount);
        uint256 stEthBal = IERC20(STETH_TOKEN).balanceOf(address(this));
        IERC20(STETH_TOKEN).approve(LIDO_CRV_POOL, stEthBal);
        uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
        IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);
        // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
        require(sent, "Failed to send Ether");
    }
```
The mentioned `maxSlippage` value is set with a 1% in the constructor and it could be modified only by the contract owner to any other value. 
```
function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

## Tools Used
Manual testing

## Recommended Mitigation Steps
Allow users to set a custom slippage when staking or unstaking. If Owner sets slippage with a low value, funds could be blocked as the minOut value wont be filled. If he sets it with a higher value then sandwich attacks could be done against the protocol when unstaking ETH.