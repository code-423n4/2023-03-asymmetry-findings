--- SfrxEth.sol ---
Line 79 -> `        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
            (10 ** 18 - maxSlippage)) / 10 ** 18;`

The maxSlippage parameter can be set by owner. there is no check for whether maxSlippage is <= 1e18. If it is = then the entire equation resolves to minOut = 0. Since this is used in the curve swap, a minAmountZero allows for trade funds to be withdrawn to be stolen as there's no infinite slippage. If it > then there will be an underflow error which would prevent users from withdrawing. The same issue can be found for the StEthEthPool trade on WstEth.sol line 61 and 

--- SafETH.sol ---
1. SafETH.sol is the owner of all the derivatives contracts. It does not have a method to renounce ownership should it ever need to be deprecated. 
Example: If the original SafETH.sol is ever rendered unusable, such as from a storage corruption problem with the Proxy contract, users would not be able to withdraw from the derivatives contract since it is the only one able to do withdrawals. The funds in the derivatives contracts would be locked forever. The SafETH file should include an onlyOwner function in which SafETH.sol transfers ownership of the derivatives contract to somewhere new. Consider using a roll based system where admins can permit different contracts deposit/withdraw which allows for further upgradability with multiple versions of controlled deposit contracts.

2. adjustWeight does not include a call to `rebalanceToWeights()` after the weights are adjusted. If a user backruns a change in weights without adjusting the balance then the `balance()` of a derivative may not match its expected value with weight. `rebalanceToWeights()` should be called automatically at the end after weights are adjusted.

3. tag `nonReentrant` onto the `stake` and `unstake` method for safety

--- Reth.sol ---

1. Consider using the CRV pool for RETH/ETH rather than the Uniswap V3 pool. The stableswap invariant makes it harder to engage in oracle manipulation and results in less slippage at high volumes. It also has ~$2M more in liquidity then the Uniswap Pool resulting in a lower price impact

--- All derivatives contracts ---
All of the derivatives contracts are OwnableUpgradable. However, ownableUpgradable does not set the owner of the contract without an explicit call. It requires you to call `__Ownable_init()` to transfer ownership of the contract to msg.sender. After the constructor there is no ownership set. Similasrly, the initializer modifier does not do a check that the caller is authorized to do so. As a result anyone can backrun the contract deployment and call `initialize(address _owner)` and make themselves the owner, hijacking the contract. This can be resolved by calling `__Ownable_init()` in the constructor to set deployer as owner, and then adding the `onlyOwner` modifier to the `initialize()` function.
