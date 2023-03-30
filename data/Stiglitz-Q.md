## Unchecked return (does not contain publicly known issues)
In the following lines of code are unchecked return values. Several ERC20 tokens have unusual behavior. Also, externally called contract codes can be changed when using an upgradable pattern, so checking return values is a good practice.

**SfrxEth.sol**
- Line #61 `IsFrxEth(SFRX_ETH_ADDRESS).redeem(...)`
- Line #77 `IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).exchange(...)`

**WstEht.sol**
- Line #57 `IWStETH(WST_ETH).unwrap(_amount)`
- Line #61 `IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut)`
