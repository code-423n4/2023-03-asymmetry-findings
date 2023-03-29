## Setters should check that the new value is different than the current storage value
These functions does not assert that the value intended to be written in the storage is not similar to the current storage value, leading to unnecessary gas consumption if the value is already set.

### Context
SafEth.sol:setPauseUnstaking()
SafEth.sol:setPauseStaking()
SafEth.sol:setMaxAmount()
SafEth.sol:setMinAmount()
SafEth.sol:setMaxSlippage()

### Mitigation
Assert that the value intended to be written in the storage is not similar to the current storage value and revert 
if they are similar in order to not spend unnecessary gas on a SSTORE operations.

### Tools used
Manual review / gas reporter

---

## Cache array value outside of loops to save on MLOADs/SLOADs
Caching variables outside of loops may save hundreds of units of gas by requiring less MLOADs/SLOADs at each iteration.

### Context
Global

### Mitigation

Example with the `stake()` function of the `SafEth.sol`contract:

```solidity
function stake() external payable {
  require(pauseStaking == false, "staking is paused");
  require(msg.value >= minAmount, "amount too low");
  require(msg.value <= maxAmount, "amount too high");

  uint256 underlyingValue;
  uint256 totalStakeValueEth;
  uint256 weight;
  uint256 ethAmount;
  uint256 depositAmount;
  uint256 derivativeReceivedEthValue;
  uint256 preDepositPrice;
	uint256 totalSupply = totalSupply();
  IDerivative derivative;
  
  for (uint256 i; i < derivativeCount; i++) {
      derivative = derivatives[i];
      underlyingValue +=
          (derivative.ethPerDerivative(derivative.balance()) * derivative.balance()) / 10 ** 18;
  }
      
  if (totalSupply == 0)
      preDepositPrice = 10 ** 18;
  else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

  
  for (uint256 i = ; i < derivativeCount; i++) {
      weight = weights[i];
     
      if (weight == 0) continue;

      ethAmount = (msg.value * weight) / totalWeight;
      derivative = derivatives[i];
      depositAmount = derivative.deposit{value: ethAmount}();
      derivativeReceivedEthValue = (derivative.ethPerDerivative(depositAmount) * depositAmount) / 10 ** 18;
      totalStakeValueEth += derivativeReceivedEthValue;
  }

  uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
  _mint(msg.sender, mintAmount);
  emit Staked(msg.sender, msg.value, mintAmount);
}
```

### Tools used
Manual review / gas reporter

---

## Unnecessary looping
A loop is used to compute the `localTotalWeight` and update the state `totalWeight` storage variable with the newly computed value.
Unlike `adjustWeight()`, there is no need to recompute the total weight as this function is supposed to only ADD a new derivative, 
so its weight could just be added to the `totalWeight` state variable value.

### Context
SafEth.sol:addDerivative()

### Mitigation
The loop can be avoided by adding the current derivative weight to the `totalWeight` storage variable, saving gas on loop's MLOADs/MSTOREs and SLOADs/SSTOREs operations.

```solidity
function addDerivative(
    address _contractAddress,
    uint256 _weight
) external onlyOwner {
    derivatives[derivativeCount] = IDerivative(_contractAddress);
    weights[derivativeCount] = _weight;
    derivativeCount++;

    totalWeight += _weight;

    emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
}
```

### Tools used
Manual review / gas reporter