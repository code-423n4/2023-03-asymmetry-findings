```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L79-L81

        if (totalSupply != 0)
            preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
        else 
            preDepositPrice = 10 ** 18; // initializes with a price of 1
```

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L72-L75

        for (uint i = 0; i < derivativeCount; i++){
            IDerivative derivative = derivatives[i];
            underlyingValue +=
                (derivative.ethPerDerivative(derivative.balance()) *
                    derivative.balance()) /
                10 ** 18;
        }

```
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L65-L66

        require(msg.value > minAmount, "amount too low");
        require(msg.value < maxAmount, "amount too high");
```
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215
    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        if (poolCanDeposit(_amount))
            return
                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
        else return poolPrice();
    }
```