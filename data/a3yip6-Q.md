## [L-01] Revert may happen if owner invoke `rebalanceToWeights` with too many tokens in derivatives
The function `rebalanceToWeights` is used by the owner to rebalence the weights of derivatives. The function would withdraw and deposit to each derivatives, which will exchange tokens with `maxSlippage`. Thus, the huge amounts of tokens in derivatives would make exchange fail because of exceeding `maxSlippage`, and the owner would fail to reblance the weights.

```
File: /contracts/SafEth.sol

function rebalanceToWeights() external onlyOwner {
    uint256 ethAmountBefore = address(this).balance;
    for (uint i = 0; i < derivativeCount; i++) {
        if (derivatives[i].balance() > 0)
            derivatives[i].withdraw(derivatives[i].balance());
    }
    uint256 ethAmountAfter = address(this).balance;
    uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;

    for (uint i = 0; i < derivativeCount; i++) {
        if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
        uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
            totalWeight;
        // Price will change due to slippage
        derivatives[i].deposit{value: ethAmount}();
    }
    emit Rebalanced();
}
```

```
File: /contracts/WstEth.sol

function withdraw(uint256 _amount) external onlyOwner {
    // burn WStETH and withdraw stETH
    IWStETH(WST_ETH).unwrap(_amount);
    uint256 stEthBal = IERC20(STETH_TOKEN).balanceOf(address(this));
    IERC20(STETH_TOKEN).approve(LIDO_CRV_POOL, stEthBal);
    uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
    // exchange stETH to ETH
    IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);
    // solhint-disable-next-line
    (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
        ""
    );
    require(sent, "Failed to send Ether");
}

```