
---------------------------

**Gas Optimization**

----------

| S No. | Issue | Instances | Gas Savings (from provided tests) |
|-----|-----|-----|-----|
| [G-01] | x += y costs more gas than x = x + y for state variables | 4 | 452  
| [G-02] | Don't compare boolean expressions to boolean literals | 2 | 110

-------------

## G-01 x += y costs more gas than x = x + y for state variables

Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**.

_There are 4 instances of this issue_

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

```
File: SafEth/SafEth.sol

72-75: underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                    derivatives[i].balance()) /
                10 ** 18;
95: totalStakeValueEth += derivativeReceivedEthValue;
172: localTotalWeight += weights[i];
192: localTotalWeight += weights[i]; 
```

-------

## G-02 Don't compare boolean expressions to boolean literals

Comparing to a constant (`true` or `false`) is a bit more expensive than directly checking the returned boolean value. I suggest using `if(!directValue)` instead of `if(directValue == false)`

`if (<x> == true)` => `if (<x>)`, `if (<x> == false)` => `if (!<x>)`

Gas Saved: **55**

_There are 2 instances of this issue_

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

```
File: SafEth/SafEth.sol

64: require(pauseStaking == false, "staking is paused");
109: require(pauseUnstaking == false, "unstaking is paused");
```

------------

