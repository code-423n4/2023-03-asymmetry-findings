## [G-01] Updating weight through adding new weight and subtracting old weight to save gas
Rather than loading storage `weights` repeatedly, updating weight directly by adding new weight and subtracting old weight can save gas. There are two instances in `adjustWeight` and `addDerivative` functions.
```
File: /contracts/SafEth.sol

// Before 
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

// After
function adjustWeight(
    uint256 _derivativeIndex,
    uint256 _weight
) external onlyOwner {
    uint256 oldWeight = weights[_derivativeIndex];
    totalWeight = totalWeight + _weight - oldWeight;
    weights[_derivativeIndex] = _weight;
    emit WeightChange(_derivativeIndex, _weight);
}
```

```
// Before
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

// After
function addDerivative(
    address _contractAddress,
    uint256 _weight
) external onlyOwner {
    derivatives[derivativeCount] = IDerivative(_contractAddress);
    weights[derivativeCount] = _weight;
    derivativeCount++;

    totalWeight = totalWeight + _weight;
    emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
}
```

## [G-02] Changing the place of the calculation of `underlyingValue` to save gas
In function `stake`, the underlyingValue is only used if `totalSupply` is not equal to 0, so culculating `underlyingValue` after judgement can save gas.

```
File: /contracts/SafEth.sol

// Before
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

// After
uint256 totalSupply = totalSupply();
uint256 preDepositPrice; // Price of safETH in regards to ETH
if (totalSupply == 0)
    preDepositPrice = 10 ** 18; // initializes with a price of 1
else {
    for (uint i = 0; i < derivativeCount; i++)
    underlyingValue +=
        (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
            derivatives[i].balance()) /
        10 ** 18;
    preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
}
```