## 1. Reduce repetition of code
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L170-L173
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L190-L193
The mentioned code fragments are repeated, instead put it in a method. (Don't repeat the code, Write Code Once)

###### Recommended Solution
1 . Create a new function:
```
function recalculateTotalWeight() private {
        uint256 localTotalWeight = 0;
        for (uint256 i; i < derivativeCount; ++i)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
}
```
2 . Call the function wherever you need, for example:
```
function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;

        recalculateTotalWeight(); // <<--------- Call the function ---------------

        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    }
```

```
function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
        weights[_derivativeIndex] = _weight;
        
        recalculateTotalWeight(); // <<--------- Call the function -------------

        emit WeightChange(_derivativeIndex, _weight);
    }
```

## 2. There is no need to recalculate the value of `totalWeight` in `addDerivative`
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L190-L193
The mentioned code lines can be replaced with a single line of code.
`addDerivative` function can be changed as follows:
```
function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;

        totalWeight += _weight; // <<----------- This line should be enough ----------

        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    }
```

## 3. Inappropriate argument is passed while calling function `ethPerDerivative`
I think the argument passed to the function `ethPerDerivative` is inappropriate, i mean `derivative[i].balance()`.
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L73
```
function stake() external payable {
        // ...
        uint256 underlyingValue = 0;

        // Getting underlying value in terms of ETH for each derivative
        for (uint i = 0; i < derivativeCount; i++)
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) * // <<-------- derivative[i].balance() is passed to ethPerDerivative
                    derivatives[i].balance()) /
                10 ** 18;
          
       // ...
}
```

Let's look at `ethPerDerivative` function in Reth contract (the argument is unused in other derivative contracts)
The parameter is called `_amount`, so the Reth contract is checking the `_amount` can be deposited to the RocketPool or not ? if yes it will fetch the RETH token price from the RocketTokenRETH and otherwise it fetches the price from the Uniswap pool:
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L211-L216
```
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
     if (poolCanDeposit(_amount)) // <<------ Reth is checking all of the contract balance can be deposited or not, because the _amount is set to derivative[i].balance()
          return RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
     else 
          return (poolPrice() * 10 ** 18) / (10 ** 18);
}
```
As we see previously, the `derivative[i].balance()` is passed to this function so `_amount` is equal to Reth contract balance, but the Reth contract doesn't want to deposit as much as its balance every time `stake` function is called.
In other word, when a user calls the `stake` function, Reth contract is not checking the user's ethers can be deposited or not ?! Instead, it every time checks all of the contract balance can be deposited or not.

## Recommended Solution
`derivative[i].balance()` should be replaced with `msg.value` like this one:
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L170