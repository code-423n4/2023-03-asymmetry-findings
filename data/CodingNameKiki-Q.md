### Issues Template
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |

| Total Found Issues | 9 |
|:--:|:--:|

### Low Risk Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| L-01 | minAmount to stake shouldn't be set below the rocket pool minimum deposit, as the function stake will revert for derivative Reth | 1 |
| L-02 | The function "adjustWeight" shouldn't be possible to set an amount of weight on a non-existing derivative, e.g should only issue indexes till the current count of derivatives | 1 | 
| L-03 | An already existing address of derivative can be added with the function "addDerivative", which might lead to wrong accounting on staking | 1 | 
| L-04 | The setter function for changing the current maxSlippage for a derivative should have a upper limit, so stakers won't experience a significant loss based on a big amount of slippage | 1 |
| L-05 | There should be a way to remove derivatives and not just setting their weight as 0, as too many derivatives may lead to a greater gas consumption in the loops prior to staking and unstaking | 1 | 
| L-06 | The function "rebalanceToWeights" should not be used often or else the contracts will lose a significant amount of funds duo to slippage, by withdrawing and depositing again the funds to the derivatives | 1 | 
| L-07 | It should not be possible to set all of the derivatives weights as zero, as this issue leads to users losing their ether prior to calling the function stake | 1 |


| Total Low Risk Issues | 7 |
|:--:|:--:|

### Non-Critical Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| N-01 | The initialize function should emit the already existing events for changing the minAmount and maxAmount | 3 |


| Total Non-Critical Issues | 1 |
|:--:|:--:|

### Refactor Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| R-01 | Unstaking function could check in the beginning if the amount of "_safEthAmount" inputted is greater than zero, otherwise it should revert | 1 | 


| Total Refactor Issues | 1 |
|:--:|:--:|

# [L-01] minAmount to stake shouldn't be set below the rocket pool minimum deposit, as the function stake will only swap tokens through Uniswap which are slightly overpriced
Basically minAmount is the minimum amount of msg.value that users can provide in order to stake. Currently there are 3 derivatives controlled by the system - Reth, SfrxEth and WstEth. The problem occurs only on the Reth derivative, as rocket pool enforce a minimum deposit, so if the minAmount is below the rocket pool minimum deposit. The function will only try to convert the msg.value to WETH and perform a swap on Uniswap in order to get Reth tokens. Considering the Uniswap pool price is slightly overpriced than the normal price, it will lead to slippage of the msg.value and less minted tokens to the user.

<img width="1222" alt="Screenshot 2023-03-28 at 13 57 56" src="https://user-images.githubusercontent.com/112419701/228215104-957b20bb-c363-4e56-8609-485e5a02f923.png">

Consider applying a minimum limit so the new set minAmount isn't below it:

```solidity
function setMinAmount(uint256 _minAmount) external onlyOwner {
        require(_minAmount >= rocketDAOProtocolSettingsDeposit.getMinimumDeposit(),"");
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }
```


# [L-02] The function "adjustWeight" shouldn't be possible to set an amount of weight on a non-existing derivative, e.g should only issue indexes till the current count of derivatives

As how it is right now the function "adjustWeight" allows adjusting weights for non-existing derivatives. In my opinion the function should check the derivative count and revert if the inputted _derivativeIndex is over the general count of derivatives. Even tho this problem doesn't lead to anything bad, that l am aware of, it should be fixed to prevent some unwanted issues from occurring.

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

Consider applying a require statement which checks the number of derivatives and revert incase the _derivativeIndex is over them.

```solidity
function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
        require(derivativeCount >= _derivativeIndex, "Non-existing index");
        weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit WeightChange(_derivativeIndex, _weight);
    }
```

# [L-03] An already existing address of derivative can be added with the function "addDerivative", which might lead to wrong accounting on staking

The function "addDerivative" is used by the owner in order to add new derivatives to the system. The problem is the function doesn't check if an already existing address of derivative is added. Therefore two identical derivatives will exist and this issue might lead to the wrong accounting of the balances prior to staking and unstaking. Consider performing some sort of check, which ensures that the new derivative added to the system isn't already an existing one.

```solidity
function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;

        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    }
```

# [L-04] The setter function for changing the current maxSlippage for a derivative should have a upper limit, so stakers won't experience a significant loss based on a big amount of slippage

The setter function for changing the current maxSlippage for a derivative should have an upper limit, so stakers won't experience a significant loss based on a big amount of slippage. Currently as higher this maxSlippage is set by the owner, the higher chance the staker will experience a significant lost of funds. It would be good to have an upper limit like 5%-10%, so the maxSlippage can't be set over that.

```solidity
function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
    ) external onlyOwner {
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
    }
```

# [L-05] There should be a way to remove derivatives and not just setting their weight as 0, as too many derivatives may lead to a greater gas consumption in the loops prior to staking and unstaking

Currently there is a function for adding new derivatives to the system, but an opposite function for removing unneeded derivatives anymore is not applied. Therefore in order to remove unwanted derivative, the function "adjustWeight" is called which basically adjust its weight to 0 so no more funds will be staked there, but it doesn't remove the derivative fully. So therefore both staking and unstaking function will loop over this unneeded derivatives and will raise the gas consumption even more. In my opinion there should some sort of way to full remove an unwanted derivative instead of just setting its weight to zero.

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
```solidity
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

# [L-06] The function "rebalanceToWeights" should not be used often or else the contracts will lose a significant amount of funds duo to slippage, by withdrawing and depositing again the funds to the derivatives.

The function "rebalanceToWeights" is used by the owner in order to rebalance each derivative to resemble the weight set for it. Basically what the function does is withdrawing the amount of funds that the derivatives hold and re-deposit it back to them based on the new set weight. The problem with this function is that calling it results to slippage and shorten the overall funds in the contracts. Shorten derrative's funds means that the staker's minted tokens will get less valuable. Therefore calling this function often, will result to the stakers taking a significant lose.

```solidity
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

# [L-07] It should not be possible to set all of the derivatives weights as zero, as this issue leads to users losing their ether prior to calling the function stake

The function "adjustWeight" is used by the owner in order to apply new weight to a certain _derivativeIndex. The main problem here is that, if all of the derivatives are successfully set as zero weight. The function "stake" won't deposit the users ether to the derivatives and therefore won't mint any tokens to them. But will consume the staker's msg.value and therefore this funds will be stuck in the contract. 

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

Consider adding a check in the adjusting weight function, that totalWeights needs to be over zero otherwise the function should revert.

```solidity
function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
        weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[I];
        totalWeight = localTotalWeight;
        require(totalWeight > 0,"");
        emit WeightChange(_derivativeIndex, _weight);
    }
```

# [N-01] The initialize function should emit the already existing events for changing the minAmount and maxAmount

The function initialize is used in order to initialize values for the contract. Like seen below, the function successfully sets the minAmount and maxAmount to certain values. But forgets to emit the already existing events allocated for this changes e.g "event ChangeMinAmount(uint256 indexed minAmount);" and "event ChangeMaxAmount(uint256 indexed maxAmount);".

```solidity
function initialize(
        string memory _tokenName,
        string memory _tokenSymbol
    ) external initializer {
        ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
        _transferOwnership(msg.sender);
        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
    }
```
```solidity
event ChangeMinAmount(uint256 indexed minAmount);
event ChangeMaxAmount(uint256 indexed maxAmount);
```

# [R-01] Unstaking function could check in the beginning if the amount of "_safEthAmount" inputted is greater than zero, otherwise it should revert

The unstaking function can perform a require statement in the beginning of the function to ensure the inputted amount of safEth is greater than zero, so the function won't loop over the derivatives just to revert at the end.

```solidity
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

Consider applying a require statement to ensure the safEth amount inputted by the user is greater than zero.

```solidity
function unstake(uint256 _safEthAmount) external {
        require(pauseUnstaking == false, "unstaking is paused");
        require(_safEthAmount > 0, "Zero unstable not allowed");
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