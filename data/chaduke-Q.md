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

