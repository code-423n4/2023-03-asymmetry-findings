### [Gas-01] Function result should be cached in ```memory``` instead of calling it again and again.
```solidity
underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *  
                    derivatives[i].balance()) /
                10 ** 18;
```
Here function result of ```derivatives[i].balance()``` should be cached in ```memory``` instead of calling it again.
```Instances(2)```
```
File: contracts/SafEth/SafEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L72-L75
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L141-L142
```

### [Gas-02] Mathematic logics for ```adjustWeight()``` could be improved to save gas.
```solidity
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
Here first weight set in mapping, then pointer(program pointer) itterate over whole mapping and add their corresponding weights and finally set result(```localTotalWeight```) to ```totalWeight``` state variable.

Here multiple stateVariable read and addition occure to set just one derivative's weight

Which can preform with less cost i.e
1. Whenever we change a weight
     a. first delete that much weight from ```totalWeight```
     b. Set corresponding weight with its index in ```weights``` mapping. 
     b. then add new weight to ```totalWeight```

```solidity
 function adjustWeight(
        uint256 _derivativeIndex, 
        uint256 _weight
    ) external onlyOwner {  
+        totalWeight = totalWeight - weights[_derivativeIndex];
         weights[_derivativeIndex] = _weight;
+        totalWeight = totalWeight + _weight;

-        uint256 localTotalWeight = 0;
-       for (uint256 i = 0; i < derivativeCount; i++) 
-            localTotalWeight += weights[i]; 
-       totalWeight = localTotalWeight;
-       emit WeightChange(_derivativeIndex, _weight); 
    }
```
```Instances(1)```
```
File: contracts/SafEth/SafEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175
```

### [Gas-03] ```> 0 ``` could replace with ```!= 0```, which is more gas efficient.
```Instances(1)```
```
File: contracts/SafEth/SafEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L141
```

### [Gas-04] Absence of sanity checks for ```_derivativeIndex``` in ```adjustWeight()```
```solidity
function adjustWeight(
        uint256 _derivativeIndex, 
        uint256 _weight
    ) external onlyOwner {  
        weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)

        ......
        ......
```
There should be a check that ensure value pair for```weights[_derivativeIndex]``` mapping exists, before making further calls.

```Instances(1)```
```
File: contracts/SafEth/SafEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L169
```

### [Gas-05] Functions Guaranteed To Revert When Called By Nomal Users Can Be Marked As ```payable```
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.
```Instances(8)```
```
File: contracts/SafEth/SafEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L168
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L185
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L205
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241
```

### [Gas-06] Arithmatic operations should uncheck
```Instances(4)```
```
File: contracts/SafEth/SafEth.sol
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L95
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L172
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L188
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L192
```

### [Gas-07] Setting ```constructor``` to ```payable``` will save gas
Saves ~13 gas per interaction
```Instances(4)```

### [Gas-08] Save gas with use of the ```import``` statement
```Instances(Multiple Instances)```




