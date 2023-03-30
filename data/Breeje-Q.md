# QA Report

## Low Risk Issues
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-01] | Possible DOS in `stake` in future | 1 |
| [L-02] | `receive` function can be removed to prevent accidental sending of Eth | 3 |

| Total Low Risk Issues | 2 |
|:--:|:--:|

### [L-01] Possible DOS in `stake` in future

`stake` method in `SafEth` uses couple of for loops bounded by the derivative count. As of now there are only 3 Derivatives, so it is fine. But in future when more derivatives will be added by owner, it can possibly lead to DOS by crossing the Block limit.

*Instance:*
```solidity
File: SafEth.sol

  function stake() external payable {
        require(pauseStaking == false, "staking is paused");
        require(msg.value >= minAmount, "amount too low");
        require(msg.value <= maxAmount, "amount too high");

        uint256 underlyingValue = 0;

        // Getting underlying value in terms of ETH for each derivative
        for (uint i = 0; i < derivativeCount; i++)
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *  // @audit checkout for the Precision in Derivatives
derivatives[i].balance()) /                                                   // @audit Gas Optimization Possible
                10 ** 18;

        uint256 totalSupply = totalSupply();
        uint256 preDepositPrice; // Price of safETH in regards to ETH
        if (totalSupply == 0)
            preDepositPrice = 10 ** 18; // initializes with a price of 1
        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

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
        // mintAmount represents a percentage of the total assets in the system
        uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
        _mint(msg.sender, mintAmount);
        emit Staked(msg.sender, msg.value, mintAmount);
    }

```

### [L-02] `receive` function can be removed to prevent accidental sending of Eth

There is a `receive` method in all the Derivatives contract. Given there are payable functions in all the contracts, it makes sense to remove the `receive` function and allow to take Eth in the contract only through payable functions.