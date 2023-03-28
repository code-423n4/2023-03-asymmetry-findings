## Empty Event

The Rebalanced event has no parameters in it to emit anything. Consider adding parameters or removing it.

```
event Rebalanced();
```

Declared here: [SafEth.sol#L34](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L34)


Used here: [SafEth.sol#L154](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L154)

## Events Associated With Setter/Update Functions

Consider having events associated with setter/update functions emit both the new and old values instead of just the new value.

[SafEth.sol#L216](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L216)
```
emit ChangeMinAmount(minAmount);
```    

[SafEth.sol#L225](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L225)
```
emit ChangeMaxAmount(maxAmount);
```  

[SafEth.sol#L207](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L207)
```
emit SetMaxSlippage(_derivativeIndex, _slippage);
```  

[SafEth.sol#L174](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L174)
```
emit WeightChange(_derivativeIndex, _weight);
```  
  

## Duplicated code that could be factorized

In SfrxEth.sol, at several places we use:

```
IERC20(SFRX_ETH_ADDRESS).balanceOf(address(this));
```
[SfrxEth.sol#L98](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L98)  
[SfrxEth.sol#L102](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L102)

Whereas we could just reuse the public `balance()` method:  
[SfrxEth.sol#L122-L124](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L122-L124)  
  
  
In Reth.sol

We duplicate lines to get the reth token address at different places:  
[Reth.sol#L187-L193](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L187-L193)  
[Reth.sol#L229-L235](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L229-L235)

We could just reuse `rethAdress()` defined here: [Reth.sol#L66](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L66)

## Lock pragmas to specific compiler version

All the files are using an unlocked compiler version.
```
pragma solidity ^0.8.13;
```

Locking the pragma helps ensure that contracts do not accidentally get deployed using, for example, the latest compiler which may have higher risks of undiscovered bugs. Contracts may also be deployed by others and the pragma indicates the compiler version intended by the original authors.
It is recommended to fix a concrete compiler version (latest without security issues). 

## Unused Function Parameters

Unused function parameters should be commented out to avoid warning when compiling files.

[SfrxEth.sol#L111](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L111)

[WstEth.sol#L86](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86)

Exemple refactored:

```
function ethPerDerivative(
        uint256 /**_amount**/
) public view returns (uint256) {
```



