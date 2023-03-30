# 1. Reentrancy Vulnerability in SafEth Contract's `unstake` function
~ The `SafEth` contract may be vulnerable to reentrancy attacks due to the order of operations in the `unstake` function. An attacker can potentially call a malicious fallback function in an external contract and repeatedly withdraw funds from the `SafEth` contract before the balance is updated, leading to loss of funds.

An attacker could potentially steal all funds from the contract leading to significant financial losses for both the contract owner and any users who have staked their `ETH`.

~ Recommended Mitigation Steps: To prevent reentrancy attacks, it is recommended to perform state updates before calling external contracts. In the `unstake` function, the `_burn` statement should be called before withdrawing funds from the derivative contracts. This ensures that the attacker cannot repeatedly withdraw funds before the `_burn` statement updates the balance of the user's account in the `SafEth` contract.
e.g:
```
function unstake(uint256 _safEthAmount) external {
    require(pauseUnstaking == false, "unstaking is paused");
    uint256 safEthTotalSupply = totalSupply();
    uint256 ethAmountBefore = address(this).balance;
    
    // update state first
    _burn(msg.sender, _safEthAmount);
    
    for (uint256 i = 0; i < derivativeCount; i++) {
        // withdraw a percentage of each asset based on the amount of safETH
        uint256 derivativeAmount = (derivatives[i].balance() *
            _safEthAmount) / safEthTotalSupply;
        if (derivativeAmount == 0) continue; // if derivative empty ignore
        derivatives[i].withdraw(derivativeAmount);
    }
    
    uint256 ethAmountAfter = address(this).balance;
    uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;
    // solhint-disable-next-line
    (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
        ""
    );
    require(sent, "Failed to send Ether");
    emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
}
```
# 2. Incorrect Precedence in `safETH` Contract `stake()` Function

~ The `safETH` contract's `stake()` function contains an incorrect order of operations, which could lead to incorrect calculation of the minted amount of `safETH` tokens.

This vulnerability can cause a loss of funds for users who stake ETH in the `safETH` contract. If the minted amount of `safETH` tokens is incorrectly calculated, it can result in an imbalance between the actual value held by the contract and the value represented by the minted `safETH` tokens.

~ Recommended Mitigation Steps: The contract should be updated to ensure that the order of operations in the calculation of the minted `safETH` tokens is correct. Specifically, the calculation of the pre-deposit price should be performed after the `totalStakeValueEth` variable is calculated. This can be achieved by moving the calculation of `preDepositPrice` after the loop that calculates `totalStakeValueEth`, as shown in the following example code:
```
    function stake() external payable {
        require(pauseStaking == false, "staking is paused");
        require(msg.value >= minAmount, "amount too low");
        require(msg.value <= maxAmount, "amount too high");

        uint256 underlyingValue = 0;

        // Getting underlying value in terms of ETH for each derivative
        for (uint i = 0; i < derivativeCount; i++)
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                    derivatives[i].balance()) /
                10 ** 18;

        uint256 totalSupply = totalSupply();
        uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
        for (uint i = 0; i < derivativeCount; i++) {
            uint256 weight = weights[i];
            IDerivative derivative = derivatives[i];
            if (weight == 0) continue;
            uint256 ethAmount = (msg.value * weight) / totalWeight;

            // This is slightly less than ethAmount because slippage
            uint256 depositAmount = derivative.deposit{value: ethAmount}();
            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
                depositAmount
            ) * depositAmount) / 10 ** 18;
            totalStakeValueEth += derivativeReceivedEthValue;
        }
        uint256 preDepositPrice; // Price of safETH in regards to ETH
        if (totalSupply == 0)
            preDepositPrice = 10 ** 18; // initializes with a price of 1
        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
        
        // mintAmount represents a percentage of the total assets in the system
        uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
        _mint(msg.sender, mintAmount);
        emit Staked(msg.sender, msg.value, mintAmount);
}
```
Additionally you can use ReentrancyGuard for `stake` and `unstake` function.

# 3. Update almost all contracts with the latest version of Solidity.