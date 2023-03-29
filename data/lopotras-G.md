# Lines of code

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L108-L120

# Description

In `unstake` function of `SafEth` contract in case user enters `_safEthAmount` bigger than his SafEth token balance, unstaking transaction will revert on `_burn`, while performing all withdrawals first.

Moving the `_burn` line higher up in the function will limit the gas spent on such failed transactions.

```
function unstake(uint256 _safEthAmount) external {
        require(pauseUnstaking == false, "unstaking is paused");
+       _burn(msg.sender, _safEthAmount);
        uint256 safEthTotalSupply = totalSupply();
        uint256 ethAmountBefore = address(this).balance;

        for (uint256 i = 0; i < derivativeCount; i++) {
            // withdraw a percentage of each asset based on the amount of safETH
            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
            if (derivativeAmount == 0) continue; // if derivative empty ignore
            derivatives[i].withdraw(derivativeAmount);
        }
-       _burn(msg.sender, _safEthAmount);
```