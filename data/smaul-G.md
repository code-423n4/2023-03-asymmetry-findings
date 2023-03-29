# Instead reading storage variable every time store it in memory variable
[SafEth.sol#L73](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L73) , [SafEth.sol#L74](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L74) , [SafEth.sol#L115](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L115) , [SafEth.sol#L118](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L118)

At Line 73 & 74  `derivatives[i]` value is fetched from storage and used directly in the for loop for 3 times that's mean 3 SLOAD call which is expensive as SLOAD cost minimum 100 Gas, instead we can store `derivatives[i]` value in a memory variable and use this value which will be MLOAD call and it's only cost 3 Gas.

To save Gas implement the code like below:
```solidity
       for (uint i = 0; i < derivativeCount; i++)
            IDerivative derivativeValue = derivatives[i];
            underlyingValue +=
                (derivativeValue.ethPerDerivative(derivativeValue.balance()) *
                    derivativeValue.balance()) /
                10 ** 18;
```

This same optimization can be applied similarly into Line 115 & 118.


# Calling `balance()` function two times which can be expensive 
[SafEth.sol#L73](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L73) , [SafEth.sol#L74](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L74)

At line 73 & 74 `balance()` function is called two times which will be costing more Gas as their is many internal & external function calls and that can be optimized by storing `balance()` function value in a memory variable and use it when it's required.


To save Gas implement the code like below:
```solidity
        for (uint i = 0; i < derivativeCount; i++)
            uint256 derivativeBalance = derivatives[i].balance();
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivativeBalance) *
                    derivativeBalance) /
                10 ** 18;
```