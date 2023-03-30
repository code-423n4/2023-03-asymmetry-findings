# 1. Lack of input validation in `WstEth` contract can lead to unexpected results
### Link: https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L76
In the `deposit` function, there is no validation done on the `msg.value` parameter before being used in the `WST_ETH.call{value: msg.value}("")` statement. This is a potential vulnerability as an attacker can manipulate the `msg.value` parameter to execute a reentrancy attack and drain funds from the contract. A possible fix for this would be to add a `require` statement to validate the value of `msg.value` before its use, like 
`require(msg.value > 0, "Deposit amount should be greater than 0");`.

# 2. Vulnerability in `Reth` Contract for RocketPool
### Link: https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L177 
### Summary

The `Reth` contract from RocketPool platform is vulnerable to a critical vulnerability which could allow an attacker to steal funds by exploiting the function `deposit()`.

### Impact
An attacker can exploit this vulnerability to steal funds from the contract. Specifically, the vulnerability lies in the `deposit()` function. If the `poolCanDeposit()` function returns false, the contract will call `swapExactInputSingleHop()` to convert `ETH` into `rETH` using `Uniswap`. However, if the conversion fails due to insufficient liquidity or other issues, the remaining `ETH` will remain in the contract and can be withdrawn by anyone.

### Recommendation 

The recommended solution is to add a check that ensures all `ETH` has been converted to `rETH` before the function exits. This can be done by adding a single line of code after calling `swapExactInputSingleHop()`:

```require(IERC20(rethAddress()).balanceOf(address(this)) > 0, "Failed to convert all ETH to rETH");```



### Example
```
function deposit() external payable onlyOwner returns (uint256) {
    // Per RocketPool Docs query addresses each time it is used
    address rocketDepositPoolAddress = RocketStorageInterface(
        ROCKET_STORAGE_ADDRESS
    ).getAddress(
            keccak256(
                abi.encodePacked("contract.address", "rocketDepositPool")
            )
        );

    RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(
            rocketDepositPoolAddress
        );

    if (!poolCanDeposit(msg.value)) {
        uint rethPerEth = (10 ** 36) / poolPrice();

        uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
            ((10 ** 18 - maxSlippage))) / 10 ** 18);

        IWETH(W_ETH_ADDRESS).deposit{value: msg.value}();
        uint256 amountSwapped = swapExactInputSingleHop(
            W_ETH_ADDRESS,
            rethAddress(),
            500,
            msg.value,
            minOut
        );

        require(IERC20(rethAddress()).balanceOf(address(this)) > 0, "Failed to convert all ETH to rETH");

        return amountSwapped;
    } else {
        address rocketTokenRETHAddress = RocketStorageInterface(
            ROCKET_STORAGE_ADDRESS
        ).getAddress(
                keccak256(
                    abi.encodePacked("contract.address", "rocketTokenRETH")
                )
            );
        RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(
            rocketTokenRETHAddress
        );
        uint256 rethBalance1 = rocketTokenRETH.balanceOf(address(this));
        rocketDepositPool.deposit{value: msg.value}();
        uint256 rethBalance2 = rocketTokenRETH.balanceOf(address(this));
        require(rethBalance2 > rethBalance1, "No rETH was minted");
        uint256 rethMinted = rethBalance2 - rethBalance1;
        return (rethMinted);
    }
}
```

# 3. Use SafeMath in your contracts for avoid integer overflow or underflow.

# 4. Missing input validation in `WstEth` contract could allow potential DoS (Denial of Service) attack
### Link: https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56-L85
### Summary
The `WstEth` contract is designed to facilitate the conversion of `wstETH` tokens into `ETH`. The contract has a function `deposit()` which allows the owner to deposit `ETH` into the contract, and a function `withdraw()` which allows the owner to convert `wstETH` tokens back to `ETH`. However, the contract does not perform any input validation on the values passed to these functions. As a result, an attacker could potentially supply invalid input values that lead to unintended behavior or cause the contract to consume more gas than expected, leading to a denial-of-service attack.
### Impact 
An attacker could potentially exploit this vulnerability to disrupt the normal functioning of the contract by, for example, supplying a large value to the `deposit()` function, causing the contract to run out of gas and fail. Alternatively, an attacker could send malformed input values to the `withdraw()` function, resulting in the contract accepting invalid input values and failing to complete the conversion process, thereby losing funds.
### Recommendation
To mitigate this risk, the contract should perform input validation to ensure that the input values to both the `deposit()` and `withdraw()` functions are within expected ranges. This can be done using `require` statements in the code that check for valid input values, such as ensuring that the amount being deposited is greater than zero before executing the `deposit` function, or ensuring that the minimum output value is less than the maximum slippage when calling the exchange function. 
### Example
```
/**
    @notice - Owner only function to Deposit ETH into derivative
    @dev - Owner is set to SafEth contract
 */
function deposit() external payable onlyOwner returns (uint256) {
    require(msg.value > 0, "Invalid input value");
    uint256 wstEthBalancePre = IWStETH(WST_ETH).balanceOf(address(this));
    // solhint-disable-next-line
    (bool sent, ) = WST_ETH.call{value: msg.value}("");
    require(sent, "Failed to send Ether");
    uint256 wstEthBalancePost = IWStETH(WST_ETH).balanceOf(address(this));
    uint256 wstEthAmount = wstEthBalancePost - wstEthBalancePre;
    return (wstEthAmount);
}

/**
    @notice - Owner only function to Convert derivative into ETH
    @dev - Owner is set to SafEth contract
 */
function withdraw(uint256 _amount) external onlyOwner {
    require(_amount > 0, "Invalid input value");
    IWStETH(WST_ETH).unwrap(_amount);
    uint256 stEthBal = IERC20(STETH_TOKEN).balanceOf(address(this));
    IERC20(STETH_TOKEN).approve(LIDO_CRV_POOL, stEthBal);
    uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
    require(minOut > 0, "Invalid minimum output");
    IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);
    // solhint-disable-next-line
    (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
        ""
    );
    require(sent, "Failed to send Ether");
}
```

# 5. Inaccurate Calculation in `SfrxEth` Contract
### Link: https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60
### Summary
The `SfrxEth` contract has an inaccurate calculation in the `withdraw` function. The calculation is used to determine the minimum amount of `ETH` to receive when exchanging `FRX_ETH` tokens for `ETH` using a Curve pool. The calculation does not take into account the decimals of the `FRX_ETH` token, leading to an incorrect result.
### Impact
This bug can cause users to receive an incorrect amount of `ETH` when withdrawing from the `SfrxEth` contract. This can result in financial losses for users who rely on the expected amount of `ETH` received from the withdrawal.
### Recommendation
The calculation in the `withdraw` function should be updated to correctly handle the decimals of the `FRX_ETH` token. Instead of using 10 ** 18, which assumes the token has 18 decimals, the number of decimals should be obtained from the token's `decimals()` function and used in the calculation.
### Example
Assuming the `FRX_ETH` token has 6 decimals and `maxSlippage` is set to 1%, the updated calculation would be:

```
uint256 minOut = ((ethPerDerivative(_amount) * _amount) /
    (10 ** IERC20(FRX_ETH_ADDRESS).decimals())) *
    (10 ** IERC20(FRX_ETH_ADDRESS).decimals() - maxSlippage)) /
    (10 ** IERC20(FRX_ETH_ADDRESS).decimals());
```

# 6. Reentrancy vulnerability in `SafEth` contract
### Link: https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108
### Summary
The `SafEth` contract is vulnerable to reentrancy attacks, this vulnerability is caused by the use of external contracts in the `stake` and `unstake` functions. The attacker can call a malicious contract that will make recursive calls to the `SafEth` contract to drain its funds.
### Impact
An attacker can exploit this vulnerability to drain the entire balance of the `SafEth` contract, leaving all users with no assets.
### Recommendation: 
To prevent this vulnerability from being exploited, a few changes can be made in the contract design. One way to do this is by using the checks-effects-interactions pattern to ensure that external contracts cannot interfere with the state of the `SafEth` contract until all necessary checks have been completed. Additionally, it may be useful to implement a withdrawal pattern so that users can withdraw their funds in smaller amounts over time, which would limit the amount an attacker can steal in a single transaction.
### Example
Here's an example implementation of the checks-effects-interactions pattern as a possible solution to the vulnerability:

```
function unstake(uint256 _safEthAmount) external {
    require(pauseUnstaking == false, "unstaking is paused");
    uint256 safEthTotalSupply = totalSupply();
    uint256 ethAmountBefore = address(this).balance;

    for (uint256 i = 0; i < derivativeCount; i++) {
        // withdraw a percentage of each asset based on the amount of safETH
        uint256 derivativeAmount = (derivatives[i].balance() *
            _safEthAmount) / safEthTotalSupply;
        if (derivativeAmount == 0) continue; // if derivative empty ignore
        // transfer derivative to a temporary variable so that it cannot be affected by reentrancy attacks
        IDerivative tempDerivative = derivatives[i];
        (bool success, ) = address(tempDerivative).call(
            abi.encodeWithSignature("withdraw(uint256)", derivativeAmount)
        );
        require(success, "Failed to withdraw derivative");
    }
    _burn(msg.sender, _safEthAmount);
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

In this example, the `IDerivative` contract is transferred to a temporary variable before calling its `withdraw()` function, which prevents any external contracts from recursively calling the `SafEth` contract during the function execution. 

# 7. If I may provide a general information, it is recommended that the latest version of Solidity and ReentrancyGuard be used in all contracts.