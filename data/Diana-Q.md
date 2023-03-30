
| S No. | Issue | Instances |
|-----|-----|-----|
| [01] | Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions | 4
| [02] | Use scientific notation rather than exponentiation | 21
| [03] | Missing events for functions that change critical parameters | 3
| [04] | Non-library or interface files should use fixed compiler versions, not floating ones | 4
| [05] | Use named imports instead of plain 'import file.sol' | 31
| [06] | Use a more recent version of solidity | 4

-------------

## 01 Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions

See [this](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

_There are 4 instances of this issue_

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

```
File: SafEth/SafEth.sol

17-18: ERC20Upgradeable,
       OwnableUpgradeable,
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

```
File: SafEth/derivatives/Reth.sol

19: contract Reth is IDerivative, Initializable, OwnableUpgradeable {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

```
File: SafEth/derivatives/SfrxEth.sol

13: contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

```
File: SafEth/derivatives/WstEth.sol

12: contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
```

-------

## 02 Use scientific notation rather than exponentiation

Use scientific notation (eg. `1e18`) rather than exponentiation (eg. `10**18`)

Scientific notation should be used for better code readability

_There are 21 instances of this issue_

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

```
File: SafEth/SafEth.sol

54: minAmount = 5 * 10 ** 17;
55: maxAmount = 200 * 10 ** 18;
75: 10 ** 18;
80: preDepositPrice = 10 ** 18;
81: else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
94: ) * depositAmount) / 10 ** 18;
98: uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

```
File: SafEth/derivatives/Reth.sol

44: maxSlippage = (1 * 10 ** 16);
171: uint rethPerEth = (10 ** 36) / poolPrice();
173: uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
174: ((10 ** 18 - maxSlippage))) / 10 ** 18);
214: RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
215: else return (poolPrice() * 10 ** 18) / (10 ** 18);
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

```
File: SafEth/derivatives/SfrxEth.sol

38: maxSlippage = (1 * 10 ** 16);
74: uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
75: (10 ** 18 - maxSlippage)) / 10 ** 18;
113: 10 ** 18
115: return ((10 ** 18 * frxAmount)
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

```
File: SafEth/derivatives/WstEth.sol

35: maxSlippage = (1 * 10 ** 16);
60: uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
87: return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
```

-----

## 03 Missing events for functions that change critical parameters

The afunctions that change critical parameters should emit events. Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services. The alternative of directly querying on-chain contract state for such changes is not considered practical for most users/usages.

Missing events and timelocks do not promote transparency and if such changes immediately affect users’ perception of fairness or trustworthiness, they could exit the protocol causing a reduction in liquidity which could negatively impact protocol TVL and reputation.

_There are 3 instances of this issue_

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

```
File: SafEth/derivatives/Reth.sol

58: function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

```
File: SafEth/derivatives/SfrxEth.sol

51: function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

```
File: SafEth/derivatives/WstEth.sol

48: function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

### Recommended Mitigation Steps

Add events to all functions that change critical parameters.

---------

## 04 Non-library or interface files should use fixed compiler versions, not floating ones

In the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

_There are 4 instances of this issue_

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

```
File: SafEth/SafEth.sol

2: pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

```
File: SafEth/derivatives/Reth.sol

2: pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

```
File: SafEth/derivatives/SfrxEth.sol

2: pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

```
File: SafEth/derivatives/WstEth.sol

2: pragma solidity ^0.8.13;
```

-----

## 05 Use named imports instead of plain 'import file.sol'

For instance, you use regular imports such as:

[SafEth.sol#L4](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L4)

```
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```

Instead of this, use named imports such as:

```
import {IERC20.sol} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```

_This issue exists in all the In-scope contracts_. _There are 31 instances of this issue_

[SafEth.sol#L4-L11](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L4-L11) , [Reth.sol#L4-L15](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L4-L15) , [SfrxEth.sol#L4-L9](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L4-L9) , [WstEth.sol#L4-L8](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L4-L8)

--------------

## 06 Use a more recent version of solidity

_There are 4 instances of this issue_

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

```
File: SafEth/SafEth.sol

2: pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

```
File: SafEth/derivatives/Reth.sol

2: pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

```
File: SafEth/derivatives/SfrxEth.sol

2: pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

```
File: SafEth/derivatives/WstEth.sol

2: pragma solidity ^0.8.13;
```

-----------

