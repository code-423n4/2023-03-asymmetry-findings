### Table Of Issues
| ID | Details of the issue |
|:---|:------|
| Low | Owner can add a new compromised derivative |
| Low | Events exist but aren't emitted on important contract changes |
| Low | The function "rebalanceToWeights" leads to bad slippage and should not be recommended for use |
| Low | Every derivative fallback function could check the upcoming ether value is either from core contract "SafEth" or from the allocated deposit pool |
| Non | The function "ethPerDerivative" in SfrxEth and WstEth takes an inputted value, but doesn't used it anywhere |
| Non | The function "addDerivative" lacks an address zero check |
| Non | Use a more recent pragma version |
| Non | If derivative has a zero balance of funds, it should continue the loop instead of adding a zero amount of underlying value |
| Non | The three require statements in the function "stake" can be refactored into one |
| Non | Create your own import names | 
| Non | Floating pragma | 

## Low - Owner can add a compromised derivative
The function "addDerivative" is used for adding new derivatives, which will interact with the core contract SafEth. The problem here is that this function may be abused by the owner, as a result the owner is able to add a new compromised derivative which might be used for stealing user's funds.

```js
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
## Low - Events exist but aren't emitted on important contract changes
The initializing function in both the core contract and other derivative, set an amount of minAmount, maxAmount and maxSlippage.
But forgets to emit the allocated events, that this changes were made. Duo to that user won't be informed regarding this important changes.

```js
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
```js
function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }
```
## Low - The function "rebalanceToWeights" leads to bad slippage and should not be recommended for use
The function "rebalanceToWeights" is used in order to rebalance the funds of the derivatives prior to their new set weights.
But this function isn't recommended for use, as it leads to bad slippage duo to depositing the funds again into their allocated pools.

```js
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
## Low - Every derivative fallback function could check the upcoming ether value is either from core contract "SafEth" or from the allocated deposit pool
Every derivative has a fallback function, which is made to accept ether to the contract. This fallback function can be refactored in a safer way, so it can receive ether only from the core contract "SafEth" and its allocated deposit pool. 
This removes issues for forcing ether to the contract by third parties.

```js
receive() external payable {}
```
## Non - The function "ethPerDerivative" in SfrxEth and WstEth takes an inputted value, but doesn't used it anywhere
The functions "ethPerDerivative" in the derivatives SfrxEth and WstEth are called with an inputted _amount, but this value isn't used anywhere in the function. So it's unnecessary and could be removed.
```js
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
    }
```
```js
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
            10 ** 18
        );
        return ((10 ** 18 * frxAmount) /
            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
    }
```
## Non - The function "addDerivative" lacks an address zero check
The function is used in order to add new derivatives to the core contract SafEth, the function is missing a check to ensure the new derivative isn't set as address(0), as the derivative address can't be changed on a later plan.

```js
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
## Non - Use a more recent pragma version
Old version of solidity is used, consider using the new one 0.8.19. You can see what new versions offer regarding bug fixed. 

```js
pragma solidity ^0.8.13;
```
## Non - If derivative has a zero balance of funds, it should continue the loop instead of adding a zero amount of underlying value
Currently the staking function loops over all of the derivatives and calculates the underlying value of them based on their balances. If a derivative has a zero balance of funds the calculations should be skipped and the loop should continue instead of calculating everything and still adding zero value to the udnerlyinValue variable.
```js
for (uint i = 0; i < derivativeCount; i++)
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                    derivatives[i].balance()) /
                10 ** 18;
```
## Non - The three require statements in the function "stake" can be refactored into one
In the beginning the staking function performs three require statements for making sure that the staking isn't paused, that the msg.value sent by the stake is above the minAmount and below the maxAmount for staking.
This three require statements can be refactored to one - "require(pauseStaking == false && msg.value >= minAmount && msg.value <= maxAmount);"
```js
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
## Non - Create your own import names
For better readability, you should name the imports instead of using the regular ones.

```js
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../interfaces/IWETH.sol";
import "../interfaces/uniswap/ISwapRouter.sol";
import "../interfaces/lido/IWStETH.sol";
import "../interfaces/lido/IstETH.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "./SafEthStorage.sol";
import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
```
## Non - Floating pragma
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

```js
pragma solidity ^0.8.13;
```