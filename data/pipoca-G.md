## 1) Gas Saving: Use Constant Variable for Repeated Calculations (10 instances)

the value `10 ** 18` is used multiple times. Every time this calculation is executed, it requires some gas to perform the operation. By defining constants like `10 ** 18` as a constant variable at the contract level, you can save gas by avoiding the need to perform the calculation every time it's used.

Here's how you can define the constant at the contract level:

```
uint256 private constant ONE_ETH = 10 ** 18;
```

Now you can use `ONE_ETH` instead of `10 ** 18` throughout your contract, and the Ethereum Virtual Machine (EVM) will use the precomputed value instead of performing the calculation each time. This results in reduced gas consumption and a more efficient contract.

For example, you can replace the following lines in the contract:

```solidity
uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
```

With the new constant:

`
uint256 minOut = (stEthBal * (ONE_ETH - maxSlippage)) / ONE_ETH;
return IWStETH(WST_ETH).getStETHByWstETH(ONE_ETH); 
`
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L35

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L60

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L87

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L74

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L80

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L81

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L81

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L214

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215

## 2) SafEth.sol - Eliminate loop in stake() function

Instead of looping through each derivative to calculate the underlying value in the stake() function, the contract can keep track of the total underlying value separately. This way, the stake() function can simply add the new ETH deposit to the total underlying value and calculate the number of safETH tokens to mint without having to loop through each derivative.

Here's an example of how the stake() function could be optimized to reduce gas usage by minimizing the number of loops:

```
function stake() external payable {
    require(pauseStaking == false, "staking is paused");
    require(msg.value >= minAmount, "amount too low");
    require(msg.value <= maxAmount, "amount too high");

    uint256 totalSupply = totalSupply();
    uint256 preDepositPrice; // Price of safETH in regards to ETH
    uint256 totalUnderlyingValue = totalStakeValueEth;

    if (totalSupply == 0)
        preDepositPrice = 10 ** 18; // initializes with a price of 1
    else preDepositPrice = (10 ** 18 * totalUnderlyingValue) / totalSupply;

    uint256 totalStakeValueEth = totalUnderlyingValue + msg.value;

    // update each derivative with the new deposit
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
        totalUnderlyingValue += derivativeReceivedEthValue;
    }

    // mintAmount represents a percentage of the total assets in the system
    uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
    _mint(msg.sender, mintAmount);
    emit Staked(msg.sender, msg.value, mintAmount);
}
```
In this optimized version, the contract keeps track of the total underlying value in the totalUnderlyingValue variable, which is initially set to the current total value of all the derivatives. When a user stakes, the contract updates the total underlying value by adding the new ETH deposit, and then updates each derivative with the new deposit. Finally, the contract calculates the number of safETH tokens to mint based on the new total underlying value and the previous total supply.

This version only loops through the derivatives once, instead of twice as in the original version. This reduces the gas cost of the function and makes it more efficient.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63


## 3) Prefer x = x + y over x += y for Gas Cost Optimization

The SafEth contract uses the x += y syntax for arithmetic operations involving addition. While this syntax is not incorrect, using the alternative x = x + y syntax can result in marginally lower gas costs and improve the overall efficiency of the contract.

The Solidity compiler optimizes the x = x + y syntax more effectively compared to the x += y syntax. Although the gas cost difference is minimal, it is recommended to utilize the more efficient syntax in a contract to minimize the overall cost for users.


https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L72

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L95

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L172

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L192













