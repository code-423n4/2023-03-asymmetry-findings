### [Low-01] User may lost his ```ETH``` due to precision loss
Inside ```stake()``` User deposited ```ETH``` sent to different derivative contracts according to their weights.
Now the amount of eth will sent to those each derivative contract is decided by following
```solidity
for (uint i = 0; i < derivativeCount; i++) {  
            uint256 weight = weights[i];
            IDerivative derivative = derivatives[i];
            if (weight == 0) continue;
            uint256 ethAmount = (msg.value * weight) / totalWeight; 

            ......
            ......
``` 
Here calculated ```ethAmount``` may suffers from precision loss and user will get less `safETH` than he actually deserve.
```
File: contracts/SafEth/SafEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84-L88
```

### [Low-02] Old Solidity Version Used
```Instances(4)```
```
File: contracts/SafEth/SafEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2
```
```
File: contracts/SafEth/derivatives/Reth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2
```
```
File: contracts/SafEth/derivatives/SfrxEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L2
```
```
File: contracts/SafEth/derivatives/WstEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L2
```

### [Low-03] Single point of failure
The `owner` role has a single point of failure and `onlyOwner` can use critical a few functions.

`owner` role in the project:

Owner is not behind a multisig and changes are not behind a timelock.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

```
File: contracts/SafEth/SafEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol
```


### [NC-01] Use Ownable2StepUpgradeable instead of OwnableUpgradeable contract
```Instances(1)```
```
File: contracts/SafEth/SafEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L9
```

### [NC-02] Owner can Renounce Ownership.

Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The OpenZeppelin’s Ownable used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design.

Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.
```
File: contracts/SafEth/SafEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol
```


### [NC-03] Floating Pragma Used
```Instances(4)```
```
File: contracts/SafEth/SafEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2
```
```
File: contracts/SafEth/derivatives/Reth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2
```
```
File: contracts/SafEth/derivatives/SfrxEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L2
```
```
File: contracts/SafEth/derivatives/WstEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L2
```


### [NC-04] Absence of both Old and New parameter in event
```event``` only emited with newly set value, not with old value.
There should be both present, newly set one and previous old value as well. 
```Instances(5)```
```
File: contracts/SafEth/SafEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L21
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L22
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L25
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L28
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L174
```

### [NC-05] Use specific notation (e.g 1e18) rather than exponentiation (e.g 10 ** 18)
```Instances(3)```
```
File: contracts/SafEth/SafEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L54
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L55
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L98
```

### [NC-06] Absence of sanity checks
Like max and low uint limit while setting uint256 like
slipage
minAmount
maxAmount

Is it previously true or false or current value is equal with previous one for bools 
pauseStaking
pauseUnstaking

```Instances(5)```
```
File: 
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L206
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L215
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L224
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L233
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L242
```

### [NC-07] For modern and more readable code, update import usages.
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

### [NC-08] Upgradeable contract is missing a ```__gap[50]``` storage variable
__gap, is empty reserved space in storage that is put in place in Upgradeable contracts. It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments.

### [NC-09] Insufficient tests coverage
The test coverage rate of the project is 98%. Testing all functions is best practice in terms of security criteria.

