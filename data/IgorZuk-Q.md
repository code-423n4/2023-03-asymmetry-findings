# QA

## Use single DEX

Currently the derivatives use Uniswap and Curve. It'd be much cleaner and easier to work with if there was just 1 service used.

## Unify deployment routines

The code for deploying contracts is repeated in multiple places. If the mainnet deployment procedure slightly diverves the test deployments, it may go unnoticed and the tests may no longer be checking the correct setup.

## Don't derive `Initializable` for `SafEth`

It's already provided by `OwnableUpgradeable`.

