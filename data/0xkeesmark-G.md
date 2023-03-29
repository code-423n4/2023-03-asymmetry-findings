# [G-01]++i costs less gas than i++
## Lines of code
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71  
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191

## Proof of Concept
Saves 6 gas per loop.


## Recommended Mitigation Steps
```
- i++
+ ++i
```


# [G-02]Using unchecked
## Lines of code
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71  
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191

## Proof of Concept
Use unchecked for arithmetic where you are sure it won't over or underflow, saving gas costs for checks added from solidity v0.8.0.

the variable i cannot overflow because of the condition i < length.

## Recommended Mitigation Steps
Consider to use below.
```
- for (uint i = 0; i < derivativeCount; i++)
+ for (uint i = 0; i < derivativeCount;)
  // something...
+  unchecked {
+	   i++;
+	 }
  }
```


# [G-03] It costs more gas to initialize variables to zero than to let the default of zero be applied

## Lines of code
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L68
```
uint256 underlyingValue = 0;

```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L83
```
uint256 totalStakeValueEth = 0; 
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L170
```
uint256 localTotalWeight = 0;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L190
```
uint256 localTotalWeight = 0;

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71  
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191
```
for (uint256 i = 0; i < derivativeCount; i++) {
for (uint256 i = 0; i < derivativeCount; i++)
for (uint256 i = 0; i < derivativeCount; i++) {
for (uint i = 0; i < derivativeCount; i++)
for (uint i = 0; i < derivativeCount; i++) {
for (uint i = 0; i < derivativeCount; i++) {
 for (uint i = 0; i < derivativeCount; i++) {
```



## Proof of Concept
In solidity do not need to initialize variable to zero.


## Recommended Mitigation Steps
```
- uint256 localTotalWeight = 0;
+ uint256 localTotalWeight;
```

```
- for (uint256 i = 0; i < derivativeCount; i++) {
+ for (uint256 i; i < derivativeCount; i++) {
```