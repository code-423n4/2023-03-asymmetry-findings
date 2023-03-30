## [L-1] Floating pragma

There are 5 instance of this issue:

```solidity
pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEthStorage.sol#L2
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L2
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L2

https://swcregistry.io/docs/SWC-103

Recommendation: using fixed solidity version.

## [L-2] Division before multiplication

The final result may be truncated due to division before multiplication.

There are 1 instance of this issue:

```
uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) * (10 ** 18 - maxSlippage)) / 10 ** 18;
```

https://github.com/code-423n4/2023-03-neotokyo/blob/main/contracts/staking/NeoTokyoStaker.sol#L1155
https://github.com/code-423n4/2023-03-neotokyo/blob/main/contracts/staking/NeoTokyoStaker.sol#L1388

## [L-3] Unexploitable Re-Entrancy

`SafEth.sol` contract has no Re-Entrancy protection in `unstake` and `stake` functions.

There is 2 instance of this issue:

```solidity
File: SafEth.sol

function stake() external payable {

function unstake(uint256 _safEthAmount) external {
```

https://github.com/code-423n4/2023-03-neotokyo/blob/main/contracts/staking/NeoTokyoStaker.sol#L1737

Recommendation: Use Openzeppelin's or Solmate's Re-Entrancy guard.

## [L-4] Usage of `approve`

There a lot of problems with `approve` function. Best pracite is to use Openzepplin's SafeERC20 library when implement any actions with ERC20.

There are 2 instance of this issue:

```solidity
File: Reth.sol

IERC20(_tokenIn).approve(UNISWAP_ROUTER, _amountIn);

File: SfrxEth.sol

IsFrxEth(FRX_ETH_ADDRESS).approve(
	FRX_ETH_CRV_POOL_ADDRESS,
	frxEthBalance
	);

```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L90
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L69

## [NC-1] Unused imports

Importing pointless files costs gas during deployment and is a bad coding practice.

There are 2 instance of this issue:

```solidity
File: SafEth.sol

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../interfaces/IWETH.sol";
import "../interfaces/uniswap/ISwapRouter.sol";
import "../interfaces/lido/IWStETH.sol";
import "../interfaces/lido/IstETH.sol";

File: Reth.sol

import "../../interfaces/frax/IsFrxEth.sol";
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L4-L8
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L5


## [NC-2] Use proper function

Each contract uses `OwnableUpgradeable` but the constructor used `_transferOwnership(_owner)` instead of `__Ownable_init`.

There are 2 instance of this issue:

```solidity    
function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L53
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L43
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L37

## [NC-3] Unnecessary division

It makes no sense to multiply and divide by the same number. It will also save some gas.

There is 1 instance of this issue:

```solidity
else return (poolPrice() * 10 ** 18) / (10 ** 18);
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215
