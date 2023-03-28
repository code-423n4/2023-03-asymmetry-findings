## SfrxEth.sol deposit() line:94-105

```
    function deposit() external payable onlyOwner returns (uint256) {
        IFrxETHMinter frxETHMinterContract = IFrxETHMinter(
            FRX_ETH_MINTER_ADDRESS
        );
        uint256 sfrxBalancePre = IERC20(SFRX_ETH_ADDRESS).balanceOf(
            address(this)
        );
        frxETHMinterContract.submitAndDeposit{value: msg.value}(address(this));
        uint256 sfrxBalancePost = IERC20(SFRX_ETH_ADDRESS).balanceOf(
            address(this)
        );
        return sfrxBalancePost - sfrxBalancePre;
    }
```

This code can be replaced with only one line of code

```function deposit() external payable onlyOwner returns (uint256) {
    return frxETHMinterContract.submitAndDeposit{value: msg.value}(address(this));
}
```
