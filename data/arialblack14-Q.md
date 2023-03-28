# QA REPORT

### Summary of low risk issues
| Number | Issue details | Instances |
|---|---|:---:|
| [L-1](#L1) | Lock pragmas to specific compiler version. | 4
| [L-2](#L2) | It is not checked if `derivativeCount > 0` which can cause a Panic Error if something happens during deployment. | 1
| [L-3](#L3) | No input validation for the `derivativeIndex` of `adjustWeight` function. | 1

*Total: 3 issues.*

### Summary of non-critical issues
| Number | Issue details | Instances |
|---|---|:---:|
| [NC-1](#NC1) |Missing timelock for critical changes. | 14
| [NC-2](#NC2) |For modern and more readable code, update import usages. | 31
| [NC-3](#NC3) |Add parameter to emitted event. | 1
| [NC-4](#NC4) |Use of `bytes.concat()` instead of `abi.encodePacked()`. | 6

*Total: 4 issues.*

---
### Low Risk Issues

### <a id=L1>[L-1]</a> Lock pragmas to specific compiler version.

##### Description
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.

##### Recommendation
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version. [solidity-specific/locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)			

##### *Instances (4):*
File: [2023-03-asymmetry/contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2 )
```solidity
2: pragma solidity ^0.8.13;
```
File: [2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2 )
```solidity
2: pragma solidity ^0.8.13;
```
File: [2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L2 )
```solidity
2: pragma solidity ^0.8.13;
```
File: [2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L2 )
```solidity
2: pragma solidity ^0.8.13;
```
### <a id=L2>[L-2]</a> It is not checked if `derivativeCount > 0` which will cause a Panic Error if something happens during deployment.  

#### Impact
Not checking if `derivativeCount` is greater than 0, will cause a [Panic Error](https://docs.soliditylang.org/en/v0.8.17/types.html#division). 
Since this involves users getting their transactions reverted, until admin notices, the impact is high as this may cause grief to users. 
But this will only happen, if something happens during deployment (`deploy.ts` file) and the  `addDerivative` function (low probability) fails to execute properly. 
So this makes it a **LOW** severity issue.

#### Proof of Concept
When calling the `stake` function, it is not checked whether the `derivativeCount` is greater than zero. 

```solidity
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

If the `addDerivative` is not called, the previous function will be reverted because the loops on lines [71-75](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71-L75) and [84-96](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84-L96) will not execute, so the `underlyingValue` will be 0 and since
`else preDepositPrice = (10 ** 18 * 0) / totalSupply;` the `preDepositPrice` will be set to 0 too.

So `totalStakeValueEth = 0` and division by 0 for `mintAmount` on [line 98](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L98) which will cause the transaction to revert.

This check is also lacking in the following functions:
`unstake`, `rebalanceToWeights`, `adjustWeight`

#### Recommended Mitigation 

Add the missing checks.

### <a id=L3>[L-3]</a> No input validation for the `derivativeIndex` of `adjustWeight` function.

#### Description

Owner could set the `_derivativeIndex` to 0 by mistake, which would cause an error such as the above to occur.

```solidity
function adjustWeight(
    uint256 _derivativeIndex,
    uint256 _weight
) external onlyOwner {
    weights[_derivativeIndex] = _weight;
    uint256 localTotalWeight = 0;
    for (uint256 i = 0; i < derivativeCount; i++)
        localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit WeightChange(_derivativeIndex, _weight);
}
```

#### Mitigation

Add a check like the following:
`require(_derivatineIndex != 0, ZeroDerivativeIndexError);`


### Non-critical Issues
### <a id=NC1>[NC-1]</a> Missing timelock for critical changes.

##### Description
A timelock should be added to functions that perform critical changes.

##### Recommendation
TODO: Add Timelock for the following functions.

##### *Instances (14):*
File: [2023-03-asymmetry/contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138 )
```solidity
138: function rebalanceToWeights() external onlyOwner {
168: ) external onlyOwner {
185: ) external onlyOwner {
205: ) external onlyOwner {
214: function setMinAmount(uint256 _minAmount) external onlyOwner {
223: function setMaxAmount(uint256 _maxAmount) external onlyOwner {
232: function setPauseStaking(bool _pause) external onlyOwner {
241: function setPauseUnstaking(bool _pause) external onlyOwner {
```
File: [2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58 )
```solidity
58: function setMaxSlippage(uint256 _slippage) external onlyOwner {
107: function withdraw(uint256 amount) external onlyOwner {
```
File: [2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51 )
```solidity
51: function setMaxSlippage(uint256 _slippage) external onlyOwner {
60: function withdraw(uint256 _amount) external onlyOwner {
```
File: [2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48 )
```solidity
48: function setMaxSlippage(uint256 _slippage) external onlyOwner {
56: function withdraw(uint256 _amount) external onlyOwner {
```
### <a id=NC2>[NC-2]</a> For modern and more readable code, update import usages.

##### Description
Solidity code is cleaner in the following way: On the principle that clearer code is better code, you should import the things you want to use. Specific imports with curly braces allow us to apply this rule better. Check out this [article](https://betterprogramming.pub/solidity-tutorial-all-about-imports-c65110e41f3a)

##### Recommendation
Import like this: `import {contract1 , contract2} from "filename.sol";`

##### *Instances (31):*
File: [2023-03-asymmetry/contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L4 )
```solidity
4: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
5: import "../interfaces/IWETH.sol";
6: import "../interfaces/uniswap/ISwapRouter.sol";
7: import "../interfaces/lido/IWStETH.sol";
8: import "../interfaces/lido/IstETH.sol";
9: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
10: import "./SafEthStorage.sol";
11: import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
```
File: [2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L4 )
```solidity
4: import "../../interfaces/IDerivative.sol";
5: import "../../interfaces/frax/IsFrxEth.sol";
6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
7: import "../../interfaces/rocketpool/RocketStorageInterface.sol";
8: import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
9: import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
10: import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
11: import "../../interfaces/IWETH.sol";
12: import "../../interfaces/uniswap/ISwapRouter.sol";
13: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
14: import "../../interfaces/uniswap/IUniswapV3Factory.sol";
15: import "../../interfaces/uniswap/IUniswapV3Pool.sol";
```
File: [2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L4 )
```solidity
4: import "../../interfaces/IDerivative.sol";
5: import "../../interfaces/frax/IsFrxEth.sol";
6: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
7: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
8: import "../../interfaces/curve/IFrxEthEthPool.sol";
9: import "../../interfaces/frax/IFrxETHMinter.sol";
```
File: [2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L4 )
```solidity
4: import "../../interfaces/IDerivative.sol";
5: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
7: import "../../interfaces/curve/IStEthEthPool.sol";
8: import "../../interfaces/lido/IWStETH.sol";
```
### <a id=NC3>[NC-3]</a> Add parameter to emitted event.

##### Description
No parameter has been passed to an emitted event. Add some parameter to better depict what has been done on the blockchain.

##### Recommendation
Consider passing a parameter to the event.

##### *Instances (1):*
File: [2023-03-asymmetry/contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L154 )
```solidity
154: emit Rebalanced();
```
### <a id=NC4>[NC-4]</a> Use of `bytes.concat()` instead of `abi.encodePacked()`.

##### Description
Rather than using `abi.encodePacked` for appending bytes, since version `0.8.4`, `bytes.concat()` is enabled.

##### Recommendation
Since version `0.8.4` for appending bytes, `bytes.concat()` can be used instead of `abi.encodePacked()`.

##### *Instances (6):*
File: [2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L70 )
```solidity
70: abi.encodePacked("contract.address", "rocketTokenRETH")
125: abi.encodePacked("contract.address", "rocketDepositPool")
136: abi.encodePacked(
162: abi.encodePacked("contract.address", "rocketDepositPool")
191: abi.encodePacked("contract.address", "rocketTokenRETH")
233: abi.encodePacked("contract.address", "rocketTokenRETH")
```