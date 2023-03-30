1. use a lot of same storage variables
- derivativeCount uses a lot of time, should load to memory first

ref: https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175
from
```
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

to
```
    function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
        weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
        uint256 _derivativeCount = derivativeCount;
        for (uint256 i = 0; i < _derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit WeightChange(_derivativeIndex, _weight);
    }
```

_________________

2. use a lot of same storage variables
ref: https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-L195
from
```
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
to
```
  function addDerivative(
    address _contractAddress,
    uint256 _weight
  ) external onlyOwner {
    uint256 _derivativeCount = derivativeCount;

    derivatives[_derivativeCount] = IDerivative(_contractAddress);
    weights[_derivativeCount] = _weight;
    derivativeCount = ++_derivativeCount;

    totalWeight += _weight;
    emit DerivativeAdded(_contractAddress, _weight, _derivativeCount);
  }
```

----------------------------------------

3. use storage value instead of memory
ref: https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214-L217
from
```
  function setMinAmount(uint256 _minAmount) external onlyOwner {
    minAmount = _minAmount;
    emit ChangeMinAmount(minAmount);
  }
```
to
```
function setMinAmount(uint256 _minAmount) external onlyOwner {
    minAmount = _minAmount;
    emit ChangeMinAmount(_minAmount);
  }
```

also the same as 
`function setMaxAmount(...)`
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223-L226

`function setPauseStaking(...)`
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232-L235

`function setPauseUnstaking(...)`
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232-L235

`function setPauseUnstaking(...)`
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241-L244

----------------------------

4. use a lot of storage variables
ref: https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63-L101
from
```
function stake() external payable {
   // Getting underlying value in terms of ETH for each derivative
   for (uint i = 0; i < derivativeCount; i++) {
     ...
   }
   
    uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
    for (uint i = 0; i < derivativeCount; i++) {
       uint256 weight = weights[i];
       ...
   }
}
```

to
```
function stake() external payable {
   // Getting underlying value in terms of ETH for each derivative
   uint _derivativeCount = derivativeCount;
   for (uint i = 0; i < _derivativeCount; i++) {
     ...
   }
   
    uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
    for (uint i = 0; i < _derivativeCount; i++) {
       uint256 weight = weights[i];
       ...
   }
}
```

-------------------
ref: https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138-L155
from
```
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
to
```
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
------------------
5. memory variable used only 1 time
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L144-L145
from
```
uint256 ethAmountAfter = address(this).balance;
uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
```
to 
```
uint256 ethAmountToRebalance = address(this).balance - ethAmountBefore;
```
