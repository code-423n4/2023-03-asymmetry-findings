## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables (`-=` too) | 4 |  452 |
| [G&#x2011;02] | Functions guaranteed to revert when called by normal users can be marked `payable` | 14 |  294 |
| [G&#x2011;03] | Setting the constructor to `payable` | 4 |  - |

Total: 22 instances over 3 issues with **746 gas** saved.


## Gas Optimizations

### [G&#x2011;01] | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables (`-=` too) 

Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**. Subtructions act the same way.

*There are 4 instances of this issue:*

```solidity
File: contracts\SafEth\SafEth.sol
72: underlyingValue += (derivatives[i].ethPerDerivative(derivatives[i].balance()) * derivatives[i].balance()) / 10 ** 18;

95: totalStakeValueEth += derivativeReceivedEthValue;

172: localTotalWeight += weights[i];

192: localTotalWeight += weights[i];

```

### [G&#x2011;02]  Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are 
`CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost.

*There are 14 instances of this issue:*

```solidity
File: contracts\SafEth\SafEth.sol
138: function rebalanceToWeights() external onlyOwner {}

165: function adjustWeight(uint256 _derivativeIndex,uint256weight)    external onlyOwner {}

182: function addDerivative(address _contractAddress,uint256 _weight) external onlyOwner {}

202: function setMaxSlippage(uint _derivativeIndex,uint _slippage) external onlyOwner {}

214: function setMinAmount(uint256 _minAmount) external onlyOwner {}

223: function setMaxAmount(uint256 _maxAmount) external onlyOwner {}

232: function setPauseStaking(bool _pause) external onlyOwner {}

241: function setPauseUnstaking(bool _pause) external onlyOwner {}
```

```solidity
File: contracts\SafEth\derivatives\WstEth.sol
48: function setMaxSlippage(uint256 _slippage) external onlyOwner {}

56: function withdraw(uint256 _amount) external onlyOwner {}
```
```solidity
File: contracts\SafEth\derivatives\SfrxEth.sol
51:  function setMaxSlippage(uint256 _slippage) external onlyOwner {}

60:  function withdraw(uint256 _amount) external onlyOwner {}
```

```solidity
File: contracts\SafEth\derivatives\Reth.sol
58:  function setMaxSlippage(uint256 _slippage) external onlyOwner {}

107:  function withdraw(uint256 amount) external onlyOwner{}
```

### [G&#x2011;03]  Setting the constructor to `payable`
You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves **13 gas on deployment** with no security risks. 

*There are 4 instances of this issue:*

```solidity
File: contracts\SafEth\SafEth.sol
38:  constructor() {
        _disableInitializers();
    }function rebalanceToWeights() external onlyOwner {}


```

```solidity
File: contracts\SafEth\derivatives\WstEth.sol
24: constructor() {
        _disableInitializers();
    }

```

```solidity
File: contracts\SafEth\derivatives\SfrxEth.sol
27:  constructor() {
        _disableInitializers();
    }

```

```solidity
File: contracts\SafEth\derivatives\Reth.sol
33:   constructor() {
        _disableInitializers();
    }

```