
# Gas Optimizations

## G-01 Cache storage value `derivativeCount` in memory before using it in a loop
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71
Storing `derivativeCount` 
##### SafeEth::stake

```solidity
File: contracts\SafeEth.sol
70:         // Getting underlying value in terms of ETH for each derivative
71:         for (uint i = 0; i < derivativeCount; i++)

```

better to do:
```solidity
        // Getting underlying value in terms of ETH for each derivative
        uint256 _derivativeCount = derivativeCount;
        for (uint i = 0; i < _derivativeCount; i++)
```

Also present here:

```solidity
File: contracts\SafeEth.sol
084:         for (uint i = 0; i < derivativeCount; i++) {
---
113:         for (uint256 i = 0; i < derivativeCount; i++) {
---
140:         for (uint i = 0; i < derivativeCount; i++) {
---
171:         for (uint256 i = 0; i < derivativeCount; i++)
---
191:         for (uint256 i = 0; i < derivativeCount; i++)

```


- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191


## G-02 x += y costs more gas than x = x + y

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L72-L74
##### SafeEth::stake
```solidity
File: contracts\SafeEth.sol
72:             underlyingValue +=
73:                 (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74:                     derivatives[i].balance()) /
75:                 10 ** 18;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L95

```solidity
File: contracts\SafeEth.sol
95:             totalStakeValueEth += derivativeReceivedEthValue;

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L172
##### SafeEth::adjustWeight
```solidity
File: contracts\SafeEth.sol
172:             localTotalWeight += weights[i];

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L192
##### SafeEth::addDerivative
```solidity
File: contracts\SafeEth.sol
192:             localTotalWeight += weights[i];

```

## G-03 Two for loops could be combined into one to save gas
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63-L101
The second loop execution do not depends either on `underlyingValue` nor `preDepositPrice`
The calculation of `underlyingValue` could be done inside the second loop, and `predepositPrice` calculated after the loop. This would save one loop but also access to the `derivatives` mapping

```solidity
File: contracts\SafeEth.sol
063:     function stake() external payable {
064:         require(pauseStaking == false, "staking is paused");
065:         require(msg.value >= minAmount, "amount too low");
066:         require(msg.value <= maxAmount, "amount too high");
067: 
068:         uint256 underlyingValue = 0;
069: 
070:         // Getting underlying value in terms of ETH for each derivative
071:         for (uint i = 0; i < derivativeCount; i++)
072:             underlyingValue +=
073:                 (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
074:                     derivatives[i].balance()) /
075:                 10 ** 18;
076: 
077:         uint256 totalSupply = totalSupply();
078:         uint256 preDepositPrice; // Price of safETH in regards to ETH
079:         if (totalSupply == 0)
080:             preDepositPrice = 10 ** 18; // initializes with a price of 1
081:         else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
082: 
083:         uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
084:         for (uint i = 0; i < derivativeCount; i++) {
085:             uint256 weight = weights[i];
086:             IDerivative derivative = derivatives[i];
087:             if (weight == 0) continue;
088:             uint256 ethAmount = (msg.value * weight) / totalWeight;
089: 
090:             // This is slightly less than ethAmount because slippage
091:             uint256 depositAmount = derivative.deposit{value: ethAmount}();
092:             uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
093:                 depositAmount
094:             ) * depositAmount) / 10 ** 18;
095:             totalStakeValueEth += derivativeReceivedEthValue;
096:         }
097:         // mintAmount represents a percentage of the total assets in the system
098:         uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
099:         _mint(msg.sender, mintAmount);
100:         emit Staked(msg.sender, msg.value, mintAmount);
101:     }

```

## G-04 Cheaper to use uint256 as boolean values

When you want to store a true/false value, the first instinct is naturally to store it as a boolean. However, because the EVM stores boolean values as a uint8 type, which takes up two bytes, it is actually more expensive to access the value. This is because it EVM words are 32 bytes long, so extra logic is needed to tell the VM to parse a value that is smaller than standard.
The most efficient way to store true/false values, then, is to use 1 for false and 2 for true.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63-L232-L244

```solidity
File: contracts\SafeEth_orig.sol
232:     function setPauseStaking(bool _pause) external onlyOwner {
233:         pauseStaking = _pause;
234:         emit StakingPaused(pauseStaking);
235:     }
---
241:     function setPauseUnstaking(bool _pause) external onlyOwner {
242:         pauseUnstaking = _pause;
243:         emit UnstakingPaused(pauseUnstaking);
244:     }

```


