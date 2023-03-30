- SafEth.sol L214 (function setMinAmount) and L223 (function setMaxAmount): there should be a test that min amount < max amount in order to not prevent user to use the contract (temporarily).

- Reth.sol L121 (function poolCanDeposit) and L158 (function deposit): the snippet to get rocketDepositPoolAddress should be factorized into a private function to avoid code duplication

- Reth.sol L187 (function deposit) and L229 (function poolPrice): the snippet to get rocketTokenRETHAddress is the exact same as the function rethAddress which should just be called to avoid code duplication.