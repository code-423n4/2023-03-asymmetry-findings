# Qualitative Analysis:
| Metric | Rating |
| :-----: | :-----: |
| Test Coverage | Great |
| Best Practices | Followed |
| Documentation | Poor |
| NatSpec Comments | Great |
| Decentralization | Not Fully Decentralized |
| Code Quality & Complexity | Excellent |

In general, the codebase appears to be impressive. It is notably absent of code complexity and boasts excellent code quality. Best practices and NatSpec have also been utilized effectively, and test coverage is commendable at over 90%. However, given the relatively small size of the codebase, achieving 100% test coverage is expected and would be beneficial for further improvements. Regrettably, there is currently no developer documentation available. It is strongly recommended that the project team prioritize the addition of documentation as soon as possible. It is also worth noting that the protocol is not fully decentralized, with main functions such as stake and unstake being pausable. This presents a risk of a central point of failure, and measures should be taken to address this potential vulnerability.

# Findings Summary:
| No | Issue |
| :-----: | :-----: |
| L-01 | Users will lose funds if `totalWeight` will remained uninitialzed |
| L-01 | No Storage Gap for Upgradeable Contract Might Lead to Storage Slot Collision |
| L-01 | Unbounded loop |
| NC-1 | Dangerous Maths - Loss of precision due to rounding |
| NC-1 | Ignored return values |
| NC-1 | Boolean Equality - Compares to a boolean |
| NC-1 | Unlocked Pragma |
| NC-1 | Unspecific Imports |
| NC-1 | Unused function parameter/argument |
| NC-1 | Solidity Function Ordering Guidelines not followed |
| BONUS | Goerli & Rinkeby deprecated |

# Low Findings:
## L-01: Users will lose funds if `totalWeight` will remained uninitialized
`totalWeight` is initiated to the default value (zero) in [SafEthStorage.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEthStorage.sol#L19)
```solidity
    uint256 public totalWeight;
```
It is then used in the [safEth.stake()](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L88) to calculate the `ethAmount`. If the owner didn't or forgets to initialize `totalWeight` with a value greater than zero. `ethAmount` calculation will also be zero but the transaction will not revert as well. So, the Users will lose all the `msg.value` sent and ethers will be frozen in the contract. 
#### Recommended Mitigation Step: 
To prevent this from happening, a check should be added in the safEth.stake() function to ensure that no one calls the function when totalWeight is zero
```solidity
    require(totalWeight != 0, "ERROR");
```
NOTE: This bug may potentially have a higher severity level, Medium or High. However, despite attempting to create a PoC for a day, I was unable to do so due to my limited experience. As a result, I'm reporting this bug with a lower severity level.

## L-02: No Storage Gap for Upgradeable Contract Might Lead to Storage Slot Collision
All contracts are intended to be upgradeable contracts in the code base. However, none of these contracts contain storage gaps. The storage gap is essential for the upgradeable contracts because "It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments". 

#### Recommended Mitigation Step: 
Add an appropriate storage gap at the end of upgradeable contracts:
```solidity
uint256[50] private __gap;
```
Reference: https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#storage-gaps

## L-03: Unbounded loop
`SafEth.sol`'s many functions' especially [stake()](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71) and [unstake()](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113) loops depend on `derivativeCount`.
Even tho `derativeCount` can be only incremented by the contract's [owner](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L188). It still can become large in the future and DoS many contract functions. Especially [stake()](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71) function, as `msg.value` it also used inside its loop. 

#### Recommended Mitigation Step: 
Consider adding an upper limit for the derativeCount or a way to decrement the `derativeCount`


# Non-Critical Issues:

## NC-1: Dangerous Maths - Loss of precision due to rounding
There are two instances of this issue:
[Reth.sol#L173](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L173):
```solidity
      uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) * ((10 ** 18 - maxSlippage))) / 10 ** 18);
```
[SfrxEth#L74](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L74):
```solidity
      uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) * (10 ** 18 - maxSlippage)) / 10 ** 18;
```
Add scalars so roundings are negligible.

## NC-2: Ignored return values
```
01. SafEth.rebalanceToWeights() (contracts/SafEth/SafEth.sol#152-169) ignores return value by derivatives[i_scope_0].deposit{value: ethAmount}() (contracts/SafEth/SafEth.sol#166)
02. Reth.swapExactInputSingleHop(address,address,uint24,uint256,uint256) (contracts/SafEth/derivatives/Reth.sol#86-105) ignores return value by IERC20(_tokenIn).approve(UNISWAP_ROUTER,_amountIn) (contracts/SafEth/derivatives/Reth.sol#93)
03. SfrxEth.withdraw(uint256) (contracts/SafEth/derivatives/SfrxEth.sol#64-92) ignores return value by IsFrxEth(SFRX_ETH_ADDRESS).redeem(_amount,address(this),address(this)) (contracts/SafEth/derivatives/SfrxEth.sol#65-69)
04. SfrxEth.withdraw(uint256) (contracts/SafEth/derivatives/SfrxEth.sol#64-92) ignores return value by IsFrxEth(FRX_ETH_ADDRESS).approve(FRX_ETH_CRV_POOL_ADDRESS,frxEthBalance) (contracts/SafEth/derivatives/SfrxEth.sol#73-76)
05. SfrxEth.withdraw(uint256) (contracts/SafEth/derivatives/SfrxEth.sol#64-92) ignores return value by IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).exchange(1,0,frxEthBalance,minOut) (contracts/SafEth/derivatives/SfrxEth.sol#81-86)
06. SfrxEth.deposit() (contracts/SafEth/derivatives/SfrxEth.sol#98-110) ignores return value by frxETHMinterContract.submitAndDeposit{value: msg.value}(address(this)) (contracts/SafEth/derivatives/SfrxEth.sol#105)
07. WstEth.withdraw(uint256) (contracts/SafEth/derivatives/WstEth.sol#58-69) ignores return value by IWStETH(WST_ETH).unwrap(_amount) (contracts/SafEth/derivatives/WstEth.sol#59)
08. WstEth.withdraw(uint256) (contracts/SafEth/derivatives/WstEth.sol#58-69) ignores return value by IERC20(STETH_TOKEN).approve(LIDO_CRV_POOL,stEthBal) (contracts/SafEth/derivatives/WstEth.sol#61)
09. WstEth.withdraw(uint256) (contracts/SafEth/derivatives/WstEth.sol#58-69) ignores return value by IStEthEthPool(LIDO_CRV_POOL).exchange(1,0,stEthBal,minOut) (contracts/SafEth/derivatives/WstEth.sol#63)
10. DerivativeMock.withdrawAll() (contracts/mocks/DerivativeMock.sol#18-42) ignores return value by IsFrxEth(SFRX_ETH_ADDRESS).redeem(balance(),address(this),address(this)) (contracts/mocks/DerivativeMock.sol#19-23)
11. DerivativeMock.withdrawAll() (contracts/mocks/DerivativeMock.sol#18-42) ignores return value by IsFrxEth(FRX_ETH_ADDRESS).approve(FRX_ETH_CRV_POOL_ADDRESS,frxEthBalance) (contracts/mocks/DerivativeMock.sol#27-30)
12. DerivativeMock.withdrawAll() (contracts/mocks/DerivativeMock.sol#18-42) ignores return value by IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).exchange(1,0,frxEthBalance,0) (contracts/mocks/DerivativeMock.sol#31-36)
```
Ensure that all the return values of the function calls are used.
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#unused-return

## NC-3: Boolean Equality - Compares to a boolean
These checks used in `SafEth.sol` functions, compares a bool to a bool. Consider refactoring these check,
From this:
```solidity
require(pauseStaking == false, "staking is paused");
```
To this:
```solidity
require(!pauseStaking, "staking is paused");
```
From this:
```solidity
require(pauseUnstaking == false, "unstaking is paused");
```
To this:
```solidity
require(!pauseUnstaking, "unstaking is paused");
```

## NC-4: Unlocked Pragma
Contracts should be deployed with the same compiler version and flag that they have been tested thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated or a newer compiler version that might introduce bugs that affect the contract system negatively.
Lock the pragma to `0.8.13` in all the contracts
```solidity
pragma solidity 0.8.13;
```

## NC-5: Unspecific Imports
Specific Named Imports improve readability. For example:
```solidity
import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```
Consider making these imports name-specific:
```solidity
2023-03-asymmetry\contracts\SafEth\SafEth.sol:
   8: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
   9: import "../interfaces/IWETH.sol";
  10: import "../interfaces/uniswap/ISwapRouter.sol";
  11: import "../interfaces/lido/IWStETH.sol";
  12: import "../interfaces/lido/IstETH.sol";
  13: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
  14: import "./SafEthStorage.sol";
  15: import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

2023-03-asymmetry\contracts\SafEth\derivatives\Reth.sol:
   6: import "../../interfaces/IDerivative.sol";
   7: import "../../interfaces/frax/IsFrxEth.sol";
   8: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
   9: import "../../interfaces/rocketpool/RocketStorageInterface.sol";
  10: import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
  11: import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
  12: import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
  13: import "../../interfaces/IWETH.sol";
  14: import "../../interfaces/uniswap/ISwapRouter.sol";
  15: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
  16: import "../../interfaces/uniswap/IUniswapV3Factory.sol";
  17: import "../../interfaces/uniswap/IUniswapV3Pool.sol";

2023-03-asymmetry\contracts\SafEth\derivatives\SfrxEth.sol:
   8: import "../../interfaces/IDerivative.sol";
   9: import "../../interfaces/frax/IsFrxEth.sol";
  10: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
  11: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
  12: import "../../interfaces/curve/IFrxEthEthPool.sol";
  13: import "../../interfaces/frax/IFrxETHMinter.sol";

2023-03-asymmetry\contracts\SafEth\derivatives\WstEth.sol:
   6: import "../../interfaces/IDerivative.sol";
   7: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
   8: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
   9: import "../../interfaces/curve/IStEthEthPool.sol";
  10: import "../../interfaces/lido/IWStETH.sol";
```

## NC-6: Unused function parameter/argument
`_amount` unused in both the following functions. COnsider removing it or commenting why.
```solidity
2023-03-asymmetry\contracts\SafEth\derivatives\SfrxEth.sol:
    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
    }

2023-03-asymmetry\contracts\SafEth\derivatives\WstEth.sol:
    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
            10 ** 18
        );
        return ((10 ** 18 * frxAmount) /
            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
    }
```

## NC-7: Solidity Function Ordering Guidelines not followed
PoC:
- [rethAddress](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L66), a private function defined before an external function [withdraw](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L107).
- [`receive()`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L244) defined at the end

Consider following the [Solidity Function ordering Style Guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) to increase the readability of the contracts.
```
- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- view and pure functions
```

# Informational findings:
## BONUS: Goerli & Rinkeby deprecated
Rinkeby Deprecated already and Goerli will be deprecated soon InShaaAllah. Consider using an active ETH testnet `Sepolia` for on-chain testing.
```ts
hardhat.config.ts:
goerli: {
      url: process.env.GOERLI_URL || "", // @audit testing on deprecated testnets
      accounts: {
        mnemonic: process.env.MNEMONIC,
      },
    },
rinkeby: {
      url: process.env.RINKEBY_URL || "", // @audit testing on deprecated testnets
      accounts: {
        mnemonic: process.env.MNEMONIC,
      },
    }
```