# Withdraw function on WstEth can be improved 

On [/contracts/SafEth/derivatives/WstEth.sol#L56](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56) there are some refactors that can be applied to improve both code quality and security. 

The current code is: 

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

Has some points that could be improved, the following refactored code shows it: 

```
function withdraw(uint256 _amount) external onlyOwner {
    require(_amount > 0 && _amount <= address(this).balance, "Invalid amount");
    IWStETH(WST_ETH).unwrap(_amount);
    uint256 stEthBal = IERC20(STETH_TOKEN).balanceOf(address(this));
    IERC20(STETH_TOKEN).approve(LIDO_CRV_POOL, stEthBal);
    uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
    IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);
    // solhint-disable-next-line
    payable(msg.sender).transfer(address(this).balance);
}
```

**1. Use require() for input validation:** Currently, there is no validation on the _amount parameter. It would be better to use a require() statement to validate that the _amount parameter is greater than 0 and less than the balance of the contract.
**2. Use transfer() instead of call():** The current code uses call() to send Ether to the owner of the contract. However, transfer() is a more secure and gas-efficient way to transfer Ether.