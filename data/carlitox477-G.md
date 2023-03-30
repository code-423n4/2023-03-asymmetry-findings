# `SafEth.stake()` can use `derivatives` and `weights` storage pointer to save gas in both for loop
```diff
    function stake() external payable {
        require(pauseStaking == false, "staking is paused");
        require(msg.value >= minAmount, "amount too low");
        require(msg.value <= maxAmount, "amount too high");

        uint256 underlyingValue = 0;
+       mapping(uint256 => IDerivative) storage _derivatives = derivatives;
+       IDerivative _derivative;
        // Getting underlying value in terms of ETH for each derivative
        for (uint i = 0; i < derivativeCount; i++)
            underlyingValue +=
-               (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
-                   derivatives[i].balance()) /
+               _derivative = _derivatives[i];
+               (_derivative.ethPerDerivative(_derivative.balance()) *
+                   _derivative.balance()) /
                10 ** 18;

        uint256 totalSupply = totalSupply();
        uint256 preDepositPrice; // Price of safETH in regards to ETH
        if (totalSupply == 0)
            preDepositPrice = 10 ** 18; // initializes with a price of 1
        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

        uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
+       mapping(uint256 => uint256) storage _weights = weights;
        for (uint i = 0; i < derivativeCount; i++) {
-           uint256 weight = weights[i];
+           uint256 weight = _weights[i];
-           IDerivative derivative = derivatives[i];
+           _derivative = _derivatives[i];
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


# `SafEth.stake()` can cache `derivativeCount` and `totalWeight` to save gas
This will reduce storage access in both for loops in each iteration
```diff
    function stake() external payable {
        require(pauseStaking == false, "staking is paused");
        require(msg.value >= minAmount, "amount too low");
        require(msg.value <= maxAmount, "amount too high");

        uint256 underlyingValue = 0;

        // Getting underlying value in terms of ETH for each derivative
+       uint256 _derivativeCount = derivativeCount;
-       for (uint i = 0; i < derivativeCount; i++)
+       for (uint i; i < _derivativeCount; unchecked{++i})
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
+       uint256 _totalWeight = totalWeight;
-       for (uint i = 0; i < derivativeCount; i++) {
+       for (uint i; i < _derivativeCount; unchecked{++i}) {
            uint256 weight = weights[i];
            IDerivative derivative = derivatives[i];
            if (weight == 0) continue;
-           uint256 ethAmount = (msg.value * weight) / totalWeight;
+           uint256 ethAmount = (msg.value * weight) / _totalWeight;

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


# `SafEth.stake()` can cache `derivatives[i].balance()`
In the first for loop, 2 external calls can be reduced to one by saving the queried value:
```diff
-   for (uint i = 0; i < derivativeCount; i++)
+   uint256 _derivative_balance;
+   for (uint i = 0; i < derivativeCount; i++){
+       uint256 _derivative_balance = derivatives[i].balance();
        underlyingValue +=
-           (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
-               derivatives[i].balance()) /
+            (derivatives[i].ethPerDerivative(_derivative_balance) *
+               _derivative_balance) /
            10 ** 18;
    }
    //..
```



# `SafEth.rebalanceToWeights()` can cache `derivatives[i].balance()`:
```diff
+   uint256 derivatives_balance;
    for (uint i = 0; i < derivativeCount; i++) {
+       derivatives_balance = derivatives[i].balance();
-       if (derivatives[i].balance() > 0)
-           derivatives[i].withdraw(derivatives[i].balance());
+       if (derivatives_balance > 0)
+           derivatives[i].withdraw(derivatives_balance);
    }
    //...

```

# `SafEth.rebalanceToWeights()` can use `derivatives` and `weights` storage pointer to save gas in both for loops:
```diff
    function rebalanceToWeights() external onlyOwner {
        uint256 ethAmountBefore = address(this).balance;
+       mapping(uint256 => IDerivative) storage _derivatives = derivatives;
        for (uint i = 0; i < derivativeCount; i++) {
-           if (derivatives[i].balance() > 0)
-               derivatives[i].withdraw(derivatives[i].balance());
+           if (_derivatives[i].balance() > 0)
+               _derivatives[i].withdraw(_derivatives[i].balance());
        }
        uint256 ethAmountAfter = address(this).balance;
        uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;

+       mapping(uint256 => uint256) storage _weights = weights;
        for (uint i = 0; i < derivativeCount; i++) {
-           if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
+           if (_weights[i] == 0 || ethAmountToRebalance == 0) continue;
-           uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
+           uint256 ethAmount = (ethAmountToRebalance * _weights[i]) /
                totalWeight;
            // Price will change due to slippage
-           derivatives[i].deposit{value: ethAmount}();
+           _derivatives[i].deposit{value: ethAmount}();
        }
        emit Rebalanced();
    }
```

# `SafEth.rebalanceToWeights()` can cache `derivativeCount` and `totalWeight` to save gas
This will avoid an storage access in each iteration of both for loops:

```diff
    function rebalanceToWeights() external onlyOwner {
        uint256 ethAmountBefore = address(this).balance;
+       uint256 _derivativeCount = derivativeCount;
-       for (uint i = 0; i < derivativeCount; i++) {
+       for (uint i; i < _derivativeCount; unchecked{++i}) {
            if (derivatives[i].balance() > 0)
                derivatives[i].withdraw(derivatives[i].balance());
        }
        uint256 ethAmountAfter = address(this).balance;
        uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;

+       uint256 _totalWeight = totalWeight;
-       for (uint i = 0; i < derivativeCount; i++) {
+       for (uint i; i < _derivativeCount; unchecked{++i}) {
            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
-               totalWeight;
+               _totalWeight;
            // Price will change due to slippage
            derivatives[i].deposit{value: ethAmount}();
        }
        emit Rebalanced();
    }
```


# `SafEth.adjustWeight(address,uint256)`: Refactor function to avoid for loop and save gas
Current function make use of a for loop to recalculate `totalWeight`, however this action can be done by just adding input `_weight` to current `totalWeight` and substracting previous weight corresponding to `_derivativeIndex`. This means:
```diff
    function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
+       uint previousWeight = weights[_derivativeIndex]
        weights[_derivativeIndex] = _weight;
-       uint256 localTotalWeight = 0;
-       for (uint256 i = 0; i < derivativeCount; i++)
-           localTotalWeight += weights[i];
-       totalWeight = localTotalWeight;
+       totalWeight = totalWeight - previousWeight + _weight;
        emit WeightChange(_derivativeIndex, _weight);
    }
```


# `SafEth.adjustWeight(address,uint256)`: Use storage pointer for  `weights` in order to save gas
```diff
    function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
+       mapping(uint256 => uint256) storage _weights = weights;
-       weights[_derivativeIndex] = _weight;
+       _weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
-           localTotalWeight += weights[i];
+           localTotalWeight += _weights[i];
        totalWeight = localTotalWeight;
        emit WeightChange(_derivativeIndex, _weight);
    }
```


# `SafEth.adjustWeight(address,uint256)`: Cache `derivativeCount` in order to save gas
`derivativeCount` is accessed multiple times in for loop check, it can be cache in order to avoid its storage access
```diff
    function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
        weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
+       uint256 _derivativeCount = derivativeCount;
-       for (uint256 i = 0; i < derivativeCount; i++)
+       for (uint256 i = 0; i < _derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit WeightChange(_derivativeIndex, _weight);
    }
```


# `SafEth.addDerivative(address,uint256)`: Refactor function to avoid for loop and save gas
Current function make use of a for loop to recalculate `totalWeight`, however this action can be done by just adding input `_weight` to current `totalWeight`. This means:
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


# `SafEth.addDerivative(address,uint256)`: Use storage pointer for  `weights` in order to save gas
```diff
    function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress);
+       mapping(uint256 => uint256) storage _weights = weights;
-       weights[derivativeCount] = _weight;
+       _weights[derivativeCount] = _weight;
        derivativeCount++;

        uint256 localTotalWeight = 0;

        for (uint256 i = 0; i < derivativeCount; i++)
-           localTotalWeight += weights[i];
+           localTotalWeight += _weight[i];
        totalWeight = localTotalWeight;
        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    }
```


# `SafEth.addDerivative(address,uint256)`: Cache `derivativeCount` in order to save gas
`derivativeCount` is accessed multiple times, it can be cache in order to avoid some storage access

```diff
    function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
+       uint256 _derivativeCount = derivativeCount;
-       derivatives[derivativeCount] = IDerivative(_contractAddress);
-       weights[derivativeCount] = _weight;
-       derivativeCount++;
+       derivatives[_derivativeCount] = IDerivative(_contractAddress);
+       weights[_derivativeCount] = _weight;
+       _derivativeCount++;
+       derivativeCount = _derivativeCount;

        uint256 localTotalWeight = 0;
-       for (uint256 i = 0; i < derivativeCount; i++)
+       for (uint256 i; i < _derivativeCount; unchecked{++i})
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
-       emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
+       emit DerivativeAdded(_contractAddress, _weight, _derivativeCount);
    }
```

# `SafEth.setMinAmount(uint256)`: Use `_minAmount` instead of `minAmount` to avoid storage access
This would avoid 1 storage access
```diff
function setMinAmount(uint256 _minAmount) external onlyOwner {
        minAmount = _minAmount;
-       emit ChangeMinAmount(minAmount);
+       emit ChangeMinAmount(_minAmount);
    }
```

# `SafEth.setMaxAmount(uint256)`: Use `_maxAmount` instead of `maxAmount` to avoid storage access
This would avoid 1 storage access
```diff
    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
        maxAmount = _maxAmount;
-       emit ChangeMaxAmount(maxAmount);
+       emit ChangeMaxAmount(_maxAmount);
    }
```

# `SafEth.setPauseStaking(bool)`: Use `_pause` instead of `pauseStaking` to avoid storage access
This would avoid 1 storage access
```diff
    function setPauseStaking(bool _pause) external onlyOwner {
        pauseStaking = _pause;
-       emit StakingPaused(pauseStaking);
+       emit StakingPaused(_pause);
    }
```

# `SafEth.setPauseUnstaking(bool)`: Use `_pause` instead of `pauseUnstaking` to avoid storage access
This would avoid 1 storage access
```diff
    function setPauseUnstaking(bool _pause) external onlyOwner {
        pauseUnstaking = _pause;
-       emit UnstakingPaused(pauseUnstaking);
+       emit UnstakingPaused(_pause);
    }
```