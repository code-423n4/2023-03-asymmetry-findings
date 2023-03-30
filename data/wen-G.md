### 1. stake() function reads derivativeCount twice.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84

### 2. rebalanceToWeights() function reads derivativeCount twice.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147

### 3. addDerivative() function reads derivativeCount twice.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182

### 4. setPauseStaking() could emit local variable(_pause) instead of storage variable(pauseStaking). 

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L234

### 5. setPauseUnstaking() could emit local variable(_pause) instead of storage variable(pauseUnstaking). 

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L243