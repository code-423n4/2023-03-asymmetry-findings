
## [G-01] Setting the `constructor` to `payable` can save gas 

Setting constructor to payable will save ~13 gas per instance.

There are 3 instance of this issue:

```diff
    constructor() {
        _disableInitializers();
    }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L24-L26
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L38-L40
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L33-L35
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L27-L29

## [G-02] Save gas with saving variables to memory

The instances below point to the second access of a storage variable within a function.

There are 2 instance of this issue:

```solidity
File: SafEth.sol
	// derivativeCount can be saved in memory
	{
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
	}

for (uint i = 0; i < derivativeCount; i++) {
	// store external call derivatives[i].balance() in memory
	underlyingValue += (derivatives[i].ethPerDerivative(derivatives[i].balance()) * derivatives[i].balance()) / 10 ** 18;
} 
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L186-L194
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L72-L75

## [G-03] Using bools for storage incurs overhead

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot’s contents, replace the bits taken up by the boolean, and then write back. This is the compiler’s defense against contract upgrades and pointer aliasing, and it cannot be disabled. Use uint256(1) and uint256(2) for true/false instead.

There are 2 instance of this issue:

```solidity
File: SafEthStorage.sol

bool public pauseStaking; // true if staking is paused
bool public pauseUnstaking; // true if unstaking is pause
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEthStorage.sol#L16-L17

## [G-04] Unnecessary cycle consumes a lot of gas

Contract use cycle to sum up all weight when adding a derivative or changing weight of a derivative. It is unnecessary and can consume a lot of gas when the number of derivatives increases.

There are 2 instance of this issue:

```solidity
File: SafEth.sol

for (uint256 i = 0; i < derivativeCount; i++) localTotalWeight += weights[i];
```

Recommendation: add or subtract weight from `totalWeight` without the cycle.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171-L172
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191-L192