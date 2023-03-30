# Non-issues

## Use single DEX

Currently the derivatives use Uniswap and Curve. It'd be much cleaner and easier to work with if there was just 1 service used.

## Unify deployment routines

The code for deploying contracts is repeated in multiple places. If the mainnet deployment procedure slightly diverves the test deployments, it may go unnoticed and the tests may no longer be checking the correct setup.

## Don't derive `Initializable` for `SafEth`

It's already provided by `OwnableUpgradeable`.

## Rely less on the Mainnet state in development

The tests are running on a forked state of Mainnet which is probably excessive, SafETH doesn't care about the vast majority of the blockchain state. It also makes testing different edge cases more difficult, instead of tweaking a mocked ecosystem the tests would need to work with real contracts. Harcoding real addresses everywhere also makes it impossible to run SafETH on a testnet.