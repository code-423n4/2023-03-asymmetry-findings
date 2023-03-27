## Mark SafEthStorage as abstract
This contract is dependent on SafEth.sol and provides read-only functionality, there is know sense in deploying it

## Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions
See this link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.
1 instance: SafEthStorage

## Use Ownable2StepUpgradeable instead of OwnableUpgradeable contract for SafEth
There is another Openzeppelin Ownable contract (Ownable2StepUpgradeable.sol) has transferOwnership function, use is more secure due to 2-stage ownership transfer.

## Lack of functions for sending randomly received ETH and ERC-20 (apart from the derivatives themselves)
In [WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56-L63) send balanceAfter - balanceBefore, but not all balance
```solidity
        uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
        uint256 balanceBefore = address(this).balance;
        IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);
        uint256 balanceAfter = address(this).balance;
        // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: balanceAfter - balanceBefore}(
            ""
        );
```
For resque ERC-20 add
```solidity
        uint256 balancePre = IERC20(STETH_TOKEN).balanceOf(address(this));
        IWStETH(WST_ETH).unwrap(_amount);
        uint256 balancePost = IERC20(STETH_TOKEN).balanceOf(address(this));
        uint256 stEthBal = balancePost - balancePre
        IERC20(STETH_TOKEN).approve(LIDO_CRV_POOL, stEthBal);
```

And by analogy provide resque for all derivatives

## Unnecessary operation
[Reth](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215)
```solidity
        else return (poolPrice() * 10 ** 18) / (10 ** 18);
```
Return poolPrice() without unnecessary mul and div