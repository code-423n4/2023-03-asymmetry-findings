Here are some gas optimization issues:

Redundant ERC20 balance check in withdraw() function.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56

```
function withdraw(uint256 _amount) external onlyOwner {
    IWStETH(WST_ETH).unwrap(_amount);
    uint256 stEthBal = IERC20(STETH_TOKEN).balanceOf(address(this)); // Redundant
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

The stEthBal variable is not used.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56

```
function withdraw(uint256 _amount) external onlyOwner {
    IWStETH(WST_ETH).unwrap(_amount);
    uint256 minOut = (_amount * (10 ** 18 - maxSlippage)) / 10 ** 18;
    IERC20(STETH_TOKEN).approve(LIDO_CRV_POOL, _amount);
    IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, _amount, minOut);
    // solhint-disable-next-line
    (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
        ""
    );
    require(sent, "Failed to send Ether");
}

```

balanceOf() not used. 
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L93

function balance() public view returns (uint256) {
    return IERC20(WST_ETH).balanceOf(address(this)); // Redundant
}
```
The code assigns wstEthBalancePre and wstEthBalancePost variables the same value as the WST_ETH balance of the contract. It's not needed because the wstETH amount is equal to the value of msg.value. The variables and the subtraction are redundant.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L73

This should be okay:
```
function deposit() external payable onlyOwner returns (uint256) {
    // solhint-disable-next-line
    (bool sent, ) = WST_ETH.call{value: msg.value}
```

