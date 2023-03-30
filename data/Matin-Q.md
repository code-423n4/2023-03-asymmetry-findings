### Low-Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | Derivative contract addresses cannot be modified | 1 |
| [L&#x2011;02] | Wrong function natspec | 1 |


Total: 2 instances over 2 issues


Note: The table above was created considering the **automatic findings** and thus, those are not included.



## Low-Risk Issues

### [L&#x2011;01]  Derivative contract addresses cannot be modified
Any wrong, deprecated, or upgraded address of the derivative contract cannot be modified in the future. The function `addDerivative()` just adds and pushes the contract address to the last index and considers its weights in every calculation.

*There is 1 instance of this issue:*

```solidity
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
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-L195

### [L&#x2011;02]  Wrong function natspec
Two functions `adjustWeight()` and `addDerivative()` has the same natspec comment in the @notice part:
```solidity
    @notice - Adds new derivative to the index fund
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L158
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L178
