It is more gas efficient to use custom errors when reverting since Solidity v0.8.4. This saves gas when reverting, but especially when deploying. Source: https://blog.soliditylang.org/2021/04/21/custom-errors/

Below are instances where this could be amended.

Line 113 of Reth.sol

```
    /**
        @notice - Convert derivative into ETH
     */
    function withdraw(uint256 amount) external onlyOwner {
        RocketTokenRETHInterface(rethAddress()).burn(amount);
        // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
        require(sent, "Failed to send Ether");    <--- Here
    }
```

Line 200 of Reth.sol

```
            uint256 rethBalance1 = rocketTokenRETH.balanceOf(address(this));
            rocketDepositPool.deposit{value: msg.value}();
            uint256 rethBalance2 = rocketTokenRETH.balanceOf(address(this));
            require(rethBalance2 > rethBalance1, "No rETH was minted");    <--- Here
            uint256 rethMinted = rethBalance2 - rethBalance1;
            return (rethMinted);
```

Line 64, 65, 66 of SafEth.sol

```
    function stake() external payable {
        require(pauseStaking == false, "staking is paused");    <--- Here
        require(msg.value >= minAmount, "amount too low");    <--- Here
        require(msg.value <= maxAmount, "amount too high");    <--- Here

        uint256 underlyingValue = 0;
```

Line 109 of SafEth.sol

```
    function unstake(uint256 _safEthAmount) external {
        require(pauseUnstaking == false, "unstaking is paused");    <--- Here
        uint256 safEthTotalSupply = totalSupply();
        uint256 ethAmountBefore = address(this).balance;
```

Line 127 of SafEth.sol

```
        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
            ""
        );
        require(sent, "Failed to send Ether");     <--- Here
        emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
    }
```

Line 87 of SfrxEth.sol

```
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
        require(sent, "Failed to send Ether");     <--- Here
    }
```

Line 66 of WstEth.sol

```
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
        require(sent, "Failed to send Ether");     <--- Here
    }
```

Line 73 of WstEth.sol

```
        (bool sent, ) = WST_ETH.call{value: msg.value}("");
        require(sent, "Failed to send Ether");     <--- Here
        uint256 wstEthBalancePost = IWStETH(WST_ETH).balanceOf(address(this));
        uint256 wstEthAmount = wstEthBalancePost - wstEthBalancePre;
```