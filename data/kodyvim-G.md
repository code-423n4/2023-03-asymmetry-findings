#Gas and QA report.
Use scientific notation to save gas
use `1e17` instead of `10 ** 17`
instances:
```
SafEth.sol#L54 minAmount = 5 * 10 ** 17;
SafEth.sol#L55 maxAmount = 200 * 10 ** 18;
SafEth.sol#L80 preDepositPrice = 10 ** 18;
SafEth.sol#L81 preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
SafEth.sol#L94 ) * depositAmount) / 10 ** 18;
SafEth.sol#L98 uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
WstEth.sol#L35 maxSlippage = (1 * 10 ** 16);
WstEth.sol#L60 uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
WstEth.sol#L87 return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
SfrxEth.sol#L38 maxSlippage = (1 * 10 ** 16);
SfrxEth.sol#L74 uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
SfrxEth.sol#L75 (10 ** 18 - maxSlippage)) / 10 ** 18;
SfrxEth.sol#L113 10 ** 18
SfrxEth.sol#L115 return ((10 ** 18 * frxAmount) /
Reth.sol#L44 maxSlippage = (1 * 10 ** 16);  
Reth.sol#L171 uint rethPerEth = (10 ** 36) / poolPrice();
Reth.sol#L173 uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
Reth.sol#L174 ((10 ** 18 - maxSlippage))) / 10 ** 18);
```

# Contracts can receive eth not meant for them.
Eth can sent directly to these contracts by anyone apart from the expected contracts.
instances of these token contract could receive eth directly from users this could results in deviations in internal accounting or users losing funds.
## instances:
Reth.sol#L244 receive() external payable {}
SfrxEth.sol#L126 receive() external payable {}
WstEth.sol#L126 receive() external payable {}
SafEth.sol#L246 receive() external payable {}

Recommendation:
suggests that if the contracts that are to send funds to a particular token contract are known then placing a require statement at the receive function to only allow those specific contracts to send eth directly.
e.g `require(msg.sender == address(WstEth) || msg.sender == address(SfrxEth))`


