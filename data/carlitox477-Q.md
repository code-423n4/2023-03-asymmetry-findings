# wstETH: If the protocol is meant to be multichain, then WST_ETH, LIDO_CRV_POOL and STETH_TOKEN should be immutable and the constructor modified
In order to not change the current code every time a deployment is done, the mentioned variables should be delcared as immutable and their values set in the constructor:

```diff
    contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
-      address public constant WST_ETH =
-        0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0;
+      address public immutable WST_ETH;
-      address public constant LIDO_CRV_POOL =
-          0xDC24316b9AE028F1497c275EB9192a3Ea0f67022;
+      address public immutable LIDO_CRV_POOL;
-       address public constant STETH_TOKEN =
-          0xae7ab96520DE3A18E5e111B5EaAb095312D7fE84;
+       address public immutable STETH_TOKEN;

        uint256 public maxSlippage;

        // As recommended by https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable
        /// @custom:oz-upgrades-unsafe-allow constructor
-       constructor() {
+       constructor(address _WST_ETH, address _LIDO_CRV_POOL, address _STETH_TOKEN) {
            _disableInitializers();
+           WST_ETH = _WST_ETH;
+           LIDO_CRV_POOL = _LIDO_CRV_POOL;
+           STETH_TOKEN = _STETH_TOKEN;
        }
```

# `WstEth.setMaxSlippage(uint256)`, `Reth.setMaxSlippage(uint256)` and `Sfrx.setMaxSlippage(uint256)`  should limit their inputs values to `1e18`
Given that `1e16` is 1%, then 100% would be `1e18`. Given that the slippage should never be greater than 100%.

This would DOS function withdraw/deposit in the different contracts:
* [wstEth](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L60)
* [Reth](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L173-L174)
* [SfrxEth](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75)

This constraint must be checked.

# `SafETH.stake()`: avoid totalSupply shadowing
Current code creates a variable which shadows `totalSupply`. It's name should be replaced by `_totalSupply`. This means:

```diff
-   uint256 totalSupply = totalSupply();
+   uint256 _totalSupply = totalSupply();
    uint256 preDepositPrice; // Price of safETH in regards to ETH
-   if (totalSupply == 0)
+   if (_totalSupply == 0)
        preDepositPrice = 10 ** 18; // initializes with a price of 1
-   else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
+   else preDepositPrice = (10 ** 18 * underlyingValue) / _totalSupply;
```

# `SafETH` Stuck ETH would be freeze forever
Current `SafETH` allows receiving ETH from anyone. In case that a user/contract mistakenly sent ETH to this `SafEth` or in case that `msg.value * weight < totalWeight` in [stake](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L88) function, these funds will be freeze forever. Given that `rebalance` function takes into account the funds at the start of it calls, and the funds after converting derivatives to ETH.

There are three ways to approach this issue:

## Option A: Stuck ETH belongs to the owner
This would mean that a `withdrawStuckETH` function should be added. Given that ETH corresponding to stake/unstake actions are immediately sent to the corresponding contract/user, this function is easy to implement

```solidity
    function withdrawStuckETH() external onlyOwner(){
        address(msg.sender).call{value: address(this).balance}("");
    }
```

## Option B: Stuck ETH belongs to the first one who realize about it
This would mean adding the same function written before without the `onlyOwner` modifier

## Option C: Consider stuck ETH when rebalanceToWeights is called
This would mean that `rebalanceToWeights` should be changed to:
```diff
    function rebalanceToWeights() external onlyOwner {
-       uint256 ethAmountBefore = address(this).balance;
        for (uint i = 0; i < derivativeCount; i++) {
            if (derivatives[i].balance() > 0)
                derivatives[i].withdraw(derivatives[i].balance());
        }
-       uint256 ethAmountAfter = address(this).balance;
-       uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
+       uint256 ethAmountToRebalance = address(this).balance;

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
It would also be advisable to remove onlyOwner modifier to allow anyone to call this function, given that there is no way for the caller to do any kind of manipulation of state variables.


# `SafEth.addDerivative(address,uint256)` and `SafEth.adjustWeight(address,uint256)` should call `SafEth.rebalanceToWeights()` in order to keep the desire balance
When `SafEth.addDerivative(address,uint256)` and `SafEth.adjustWeight(address,uint256)` are called, the intention is to modify the current distribution of ETH staked. In order to make the changes in weights effective, function `SafEth.rebalanceToWeights` should be called. To do this:
1. `SafEth.rebalanceToWeights()` visibility should be change to public
2. `SafEth.rebalanceToWeights()` should be called at the end of `SafEth.addDerivative(address,uint256)` and `SafEth.adjustWeight(address,uint256)`


# `SafEth.adjustWeight(address,uint256)` can be involuntary DOSed if enough derivatives are added
Given that to calculate the total weight a for loop is used which require access to `weights` mapping, this function can be DOSed if enough derivatives are added. To avoid this current function can be replaced for:
```diff
    function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
+       uint256 previousWeight = weights[_derivativeIndex]
        weights[_derivativeIndex] = _weight;
-       uint256 localTotalWeight = 0;
-       for (uint256 i = 0; i < derivativeCount; i++)
-           localTotalWeight += weights[i];
-       totalWeight = localTotalWeight;
+       totalWeight = totalWeight + _weight - previousWeight;
        emit WeightChange(_derivativeIndex, _weight);
    }
```

# `SafEth.addDerivative(address,uint256)` can be involuntary DOSed if enough derivatives are added
Given that to calculate the total weight a for loop is used which require access to `weights` mapping, this function can be DOSed if enough derivatives are added. To avoid this current function can be replaced for:

```diff
    function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;

-       uint256 localTotalWeight = 0;
-       for (uint256 i = 0; i < derivativeCount; i++)
-           localTotalWeight += weights[i];
-       totalWeight = localTotalWeight;
+       totalWeight += _weight;
        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    }
```


# `SafEth.setMinAmount(uint256)` should check that `_minAmount != minAmount && _minAmount <= maxAmount`
In order to emit a meaningful event it should be checked that `_minAmount != minAmount`

In order to avoid DOS `stake` function, then `_minAmount <= maxAmount` should be checked. This means:

```diff
    function setMinAmount(uint256 _minAmount) external onlyOwner {
+       require(_minAmount != minAmount,ERROR_NOT_STATE_CHANGE);
+       require(_minAmount <= maxAmount, ERROR_MIN_AMOUNT_SHOULD_BE_LEQ_THAN_MAX_AMOUNT);
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }
```

# `SafEth.setMaxAmount(uint256)` should check that `_maxAmount != maxAmount && minAmount <= _maxAmount`
In order to emit a meaningful event it should be checked that `_maxAmount != maxAmount`

In order to avoid DOS `stake` function, then `minAmount <= _maxAmount` should be checked. This means:

```diff
    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
+       require(_maxAmount != maxAmount,ERROR_NOT_STATE_CHANGE);
+       require(minAmount <= _maxAmount, ERROR_MAX_AMOUNT_SHOULD_BE_GEQ_THAN_MIN_AMOUNT);
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
    }
```


# `SafEth.setPauseStaking` should check that current `pauseStaking` value is really being change
Otherwise, current implementation would allow emitting meaningless events
```diff
function setPauseStaking(bool _pause) external onlyOwner {
        pauseStaking = _pause;
+       require(pauseStaking != _pause, ERROR_NOT_STATE_CHANGE);
        emit StakingPaused(pauseStaking);
    }
```

# `SafEth.setPauseUnstaking` should check that current `pauseUnstaking` value is really being change
Otherwise, current implementation would allow emitting meaningless events
```diff
    function setPauseUnstaking(bool _pause) external onlyOwner {
+       require(pauseUnstaking != _pause, ERROR_NOT_STATE_CHANGE);
        pauseUnstaking = _pause;
        emit UnstakingPaused(pauseUnstaking);
    }
```