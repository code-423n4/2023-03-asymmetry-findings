QA1. Once a derivative contract is added, there is no way to remove it. 

Mitigation: add a new function so that the owner can remove an existing derivative contract.

QA2. The protocol assumes that each derivative has 18 decimals, so the protocol will break when the underlying derivative has different decimals. 

For example, the following code for ``stake()`` assumes a 18 decimals for the underlying derivative:
```javascript
 uint256 underlyingValue = 0;

        // Getting underlying value in terms of ETH for each derivative
        for (uint i = 0; i < derivativeCount; i++)
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                    derivatives[i].balance()) /
                10 ** 18;
```

Mitigation: add a function decimals() for each derivative contract and use it to calculate different quantities such as ``underlyingValue``.

QA3. The ``stake()`` function has a precision loss issue due to the use of divide-before-multiplication.

(https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L63-L101](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L63-L101)

First, it calculates ``preDepositPrice`` as L81:
```javascript
preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
```

Then, it calculates the number of shares of ``safETH`` as:
```javascript
 uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```

Mitigation: 
To avoid larger precision loss and twice divisions, the correct formula should be
```javascript
 uint256 mintAmount;
if(totalSupply == 0) minAmount = (totalStakeValueEth;
else minAmount =  totalStakeValueEth * totalSupply / underlyingValue;
```

