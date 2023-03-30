# Table of contents

- [L-01] Unable to remove derivative 
- [L-02] Unstaking should not be paused
- [L-03] Function `RebalanceToWeight` may revert
- [L-04] Functions `stake` and `unstake` may revert
- [L-05] Include check to ensure sufficient funds for unstaking
- [N-01] Insufficient test coverage
- [N-02] Import declarations should import specific identifiers, rather than the whole file
- [N-03] Use a more recent version of Solidity 
- [N-04] Non-library/interface files should use fixed compiler versions, not floating ones
- [N-05] NatSpec comments should be increased in contracts

## [L-01] Unable to remove derivative 

In  `SafEth.sol`, there is no function available to remove a derivative. There should be such a function available in case a derivative should be removed.

## [L-02] Unstaking should not be paused

In general, function `unstake` should not have a pause modifier. Users should always be able to withdraw their funds. 

```
File:  2023-03-asymmetry/contracts/SafEth/SafEth.sol 

108:    function unstake(uint256 _safEthAmount) external {
109:    require(pauseUnstaking == false, "unstaking is paused");
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L109

##  [L-03] Function `RebalanceToWeight` may revert

Function `RebalanceToWeight` in `SafEth.sol` withdraws all derivatives and re-deposits them to have the correct weights. When it comes to a point where there are many derivatives added and a huge number of users, the function may revert due to reaching maximum gas limit. Consider breaking up the loop into smaller loops so that each execution can be successful.


```
File:  2023-03-asymmetry/contracts/SafEth/SafEth.sol (L138-155)

function rebalanceToWeights() external onlyOwner {
        uint256 ethAmountBefore = address(this).balance;
        for (uint i = 0; i < derivativeCount; i++) {
            if (derivatives[i].balance() > 0)
                derivatives[i].withdraw(derivatives[i].balance());
        }
        uint256 ethAmountAfter = address(this).balance;
        uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;

        for (uint i = 0; i < derivativeCount; i++) {
            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                totalWeight;
            // Price will change due to slippage
            derivatives[i].deposit{value: ethAmount}();
        }
        emit Rebalanced();
    }
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L138-L155

## [L-04] Functions `stake` and `unstake` may revert

Similar to [L-03] above, both functions `stake` and `unstake` of `SafEth.sol` iterates over the derivatives to determine the underlyingValue and derivativeAmount respectively. The function may revert due to maximum gas limit reached if there are eventually a large number of derivatives added. Consider breaking up the loop into smaller loops so that each execution can be successful.

Function `stake`:
```
File: 2023-03-asymmetry/contracts/SafEth/SafEth.sol (L63-75)

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
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L63-L75

Function `unstake`:

```
File: 2023-03-asymmetry/contracts/SafEth/SafEth.sol (L108-119)

function unstake(uint256 _safEthAmount) external {
        require(pauseUnstaking == false, "unstaking is paused");
        uint256 safEthTotalSupply = totalSupply();
        uint256 ethAmountBefore = address(this).balance;

        for (uint256 i = 0; i < derivativeCount; i++) {
            // withdraw a percentage of each asset based on the amount of safETH
            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
            if (derivativeAmount == 0) continue; // if derivative empty ignore
            derivatives[i].withdraw(derivativeAmount);
        }
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L108-L119

## [L-05] Include check to ensure sufficient funds for unstaking

In function `unstake` of `SafEth.sol`, include a check to ensure user has sufficient funds for unstaking to minimize gas loss when the function reverts due to insufficient funds. 

```
File: 2023-03-asymmetry/contracts/SafEth/SafEth.sol (L108-129)

function unstake(uint256 _safEthAmount) external {
        require(pauseUnstaking == false, "unstaking is paused");
        uint256 safEthTotalSupply = totalSupply();
        uint256 ethAmountBefore = address(this).balance;

        for (uint256 i = 0; i < derivativeCount; i++) {
            // withdraw a percentage of each asset based on the amount of safETH
            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
            if (derivativeAmount == 0) continue; // if derivative empty ignore
            derivatives[i].withdraw(derivativeAmount);
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

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L108-L129

## [N-01] Insufficient test coverage
The test coverage rate of the project is 92%. Testing all functions is best practice in terms of security criteria.

While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

## [N-02] Import declarations should import specific identifiers, rather than the whole file

Using import declarations of the form `import {<identifier_name>} from "some/file.sol"` avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation.

For example:

```
File:  2023-03-asymmetry/contracts/SafEth/SafEth.sol 

4:    import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
5:    import "../interfaces/IWETH.sol";
6:    import "../interfaces/uniswap/ISwapRouter.sol";
7:    import "../interfaces/lido/IWStETH.sol";
8:    import "../interfaces/lido/IstETH.sol";
9:    import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
10:    import "./SafEthStorage.sol";
11:    import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
```

## [N-03] Use a more recent version of Solidity 

Old version of solidity is used, consider using the new one 0.8.17.
You can see what new versions offer regarding bug fixed [here](https://github.com/ethereum/solidity/blob/develop/Changelog.md).

For example: 

```
File: 2023-03-asymmetry/contracts/SafEth/SafEth.sol 
2:    pragma solidity ^0.8.13;
```

## [N-04] Non-library/interface files should use fixed compiler versions, not floating ones

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

For example: 

```
File: 2023-03-asymmetry/contracts/SafEth/SafEth.sol 
2:    pragma solidity ^0.8.13;
```

## [N-05] NatSpec comments should be increased in contracts
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the [Solidity official documentation](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html).