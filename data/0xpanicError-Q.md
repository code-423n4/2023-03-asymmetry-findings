https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

# Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Truncation errors due to division | 2 |
### [L-1] Truncation errors due to division
Variables that represent value of ERC20 tokens should always have 18 decimal places. There is no need to divide
`underlyingValue` and `derivativeReceivedEthValue` by `10**18` as they are remultiplied in later lines anyways. Doing this
causes rounding errors which can lead to unwanted concequences.

*Instances (2)*:
```solidity
File: contracts/SafEth/SafEth.sol

72:   underlyingValue +=

73:      (derivatives[i].ethPerDerivative(derivatives[i].balance()) *

74:          derivatives[i].balance()) /

75       10 ** 18;

81:   else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;


```

```solidity
File: contracts/SafEth/SafEth.sol

92:   uint derivativeReceivedEthValue = (derivative.ethPerDerivative(

93:     depositAmount

94:   ) * depositAmount) / 10 ** 18;

95    totalStakeValueEth += derivativeReceivedEthValue;

98:   uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;

```

