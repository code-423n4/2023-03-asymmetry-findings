
# Floating pragma

## Description

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

Floating Pragma List:
[SafeEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2) - `^0.8.13`
[Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2) - `^0.8.13`
[SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol) - `^0.8.13`
[WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L2) - `^0.8.13`

## Recommendation

Consider locking the compiler pragma to the specific version of the Solidity compiler used during testing and development.


# Shadowed variables

## References

- [SafEth.sol#77](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L77)

## Description

In the `stake` function in `SafEth.sol`, a `uint256 totalSupply` variable is defined
and shadows the existing `totalSupply()` function. This can be dangerous as any following code within
that context will lose the ability to call the `totalSupply()` function.
It is best practice to avoid shadowing variables and functions.

## Recommendation 

Consider instead prefixing the local `totalSupply` variable with an underscore to avoid shadowing the `totalSupply()` function

# Users can lock funds after contract initialization

## References

- [`SafEth.sol#48-56`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L48-L56)
- [`OwnableUpgradeable#29`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/096eb7efec4ddce33ed4ffb68b436e8dc1d80e6a/contracts/access/OwnableUpgradeable.sol#L29)

## Description

In the `initialize` function for `SafEth.sol`, the `_transferOwnership()` function is invoked to transfer the ownership to the deployer. The OpenZeppelin `OwnableUpgradeable` library includes a `__Ownable_init` function which has the same functionality but is intended to be used during initialization.


## Recommendation

Consider replacing the `_transferOwnership()` call with ` __Ownable_init()` in the initialize function to adhere to best practices set by the owners of the `OwnableUpgradeable` library.

# Unused Import

## References

- [`Reth.sol#5`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/derivatives/Reth.sol#L5)

## Description

The `IsFrxEth.sol` contract is imported in `Reth.sol` but remains unused.

## Recommendation

Consider removing all unused imports in order to keep code clean and readable.

# Unnecessary Casts

## References

- [`SfrxEth.sol#61`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/derivatives/SfrxEth.sol#L61)
- [`SfrxEth.sol#98`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/derivatives/SfrxEth.sol#L98)
- [`SfrxEth.sol#102`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/derivatives/SfrxEth.sol#L102)
- [`SfrxEth.sol#112`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/derivatives/SfrxEth.sol#L112)
- [`SfrxEth.sol#123`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/derivatives/SfrxEth.sol#L123)
- [`WstEth.sol#57`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/derivatives/WstEth.sol#L57)
- [`WstEth.sol#74`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/derivatives/WstEth.sol#L74)
- [`WstEth.sol#78`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/derivatives/WstEth.sol#L78)
- [`WstEth.sol#87`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/derivatives/WstEth.sol#L87)
- [`WstEth.sol#94`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/derivatives/WstEth.sol#L94)
- [`WstEth.sol#58`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/derivatives/WstEth.sol#L58)
- [`WstEth.sol#59`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/derivatives/WstEth.sol#L59)

## Description

* To improve code clarity, the `SFRX_ETH_ADDRESS` and `FRX_ETH_ADDRESS` in `SfrxEth.sol` could be
  declared as a `IsFrxEth` type. Additionally, a `balanceOf(address)` function could be added to
  `IsFrxEth.sol`. This would remove the need for `IsFrxEth` and `IERC20` casts.

* To improve code clarity, the `WST_ETH` in `WstEth.sol` could be declared as a `IWStETH` type.
  This would remove the need for `IWStETH` and `IERC20` casts.

* To improve code clarity the `STETH_TOKEN` in `WstEth.sol` could be declared as a `IERC20` type.
  This would remove the need for `IERC20` casts.

## Recommendation

Consider saving the aforementioned addresses as their interface counterparts, as well as including any required functions or interfaces to the existing interfaces.

# Lacking checks around derivative contracts

## References

- [`SafEth.sol#182`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L182-L195)

## Description

In the [`addDerivative`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182) function of the `SafeEth` contract, there is not a check to see if the `_contractAddress` parameter is actually a contract addresss. 

Adding any deployed contract address that do not conform to the `IDerivative` interface can prevent the whole protocol from working.

## Recommendation

Consider checking for `codeAt` any derivative contract added with the `addDerivative` function, or implementing an [`ERC165` supports interface](https://eips.ethereum.org/EIPS/eip-165) to ensure only valid derivative contracts are added.

# Updates can be called on non-existent derivatives

## References

- [`SafEth#165](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L165)
- [`SafEth.sol#202`](https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L202)

## Description

Both the `adjustWeight` and `setMaxSlippage` functions can be called on non-existent derivatives.

## Recommendation

In order to restrict inputs into the contract, consider requiring the index set to be a valid derivative for both the `adjustWeight` and `setMaxSlippage` functions.

# `approve` external call result not checked

## References

[SfrxEth.sol#69-72](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L69-L72
[WstEth.sol#59](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L59
[Reth.sol#90](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L90)

## Description

The withdraw function in SfrxEth.sol calls the approve function on FRX_ETH_ADDRESS but the return value is not checked to ensure success
The withdraw function in WstEth.sol calls the approve function on STETH_TOKEN but the return value is not checked to ensure success
The swapExactInputSingleHop function in Reth.sol calls the approve function on UNISWAP_ROUTER but the return value is not checked to ensure success

## Recommendation

Consider instead prefixing the local totalSupply variable with an underscore to avoid shadowing the totalSupply() function

# Usage of `uint`

## Description

Throughout the `SafEth.sol` contract and in one place in the `Reth.sol` contract, the `uint` term is used instead of `uint256`.

## Recommendation

It is recommended to use the `uint256` keyword over the `uint` keyword for explicitness. Consider changing all instances of `uint` to `uint256` to keep the code consistent.

# Token decimals may change and break withdraw logic

## Description

In the withdraw function in `WstEth.so`l and `SfrxEth.so`l the value `10 ** 18` is hardcoded and used to calculate
the correct number of minimum tokens. This currently works as expected but the `STETH_TOKEN`
address is a proxy and as such it is possible the decimals field may change in the future which
would break the calculation and could set `minOut` to a value that is too high or too low.