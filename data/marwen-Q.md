Report 1:

The IsFrxEth interface is not being used inside this contract Reth.sol.
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L5

in the contract SafEth.sol, the interfaces IWETH.sol, ISwapRouter.sol, IWStETH.sol and IstETH.sol are imported, but their functions are not being called anywhere in the contract. 

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L5

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L6

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L7

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L8
--------------------------------------------------------------------------------------------------
Report 2: 

Unnecessary for loops in both functions "addDerivative" and "adjustWeight"
Both functions could be improved for better readability, so the correction could reduce the signicant gas price while calling the function.

My suggestion is to change the adjustWeight from: 
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
to: 

``` 
function adjustWeight(uint256 _derivativeIndex, uint256 _weight) external onlyOwner {
        totalWeight -= weights[_derivativeIndex] ;
        totalWeight += _weight ;
        weights[_derivativeIndex] = _weight;
        emit WeightChange(_derivativeIndex, _weight);
    }
``` 
and to change the function addDerivative from: 

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
 function addDerivative(address _contractAddress, uint256 _weight ) external onlyOwner {
            derivatives[derivativeCount] = IDerivative(_contractAddress);
            weights[derivativeCount] = _weight;
            derivativeCount++; 
            totalWeight +=  _weight ;
            emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    }
``` 