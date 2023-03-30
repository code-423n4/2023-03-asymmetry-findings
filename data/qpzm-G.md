## Lines of code
[2023-03-asymmetry/SfrxEth.sol at 44b5cd94ebedc187a08884a7f685e950e987261c · code-423n4/2023-03-asymmetry · GitHub](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L115-L116)

## Vulnerability details
### Impact
`ethPerDerivative`  must get the price of derivative in terms of ETH. However, it returns `frxETHAmount / frxETHPriceInETH` .

### Proof of Concept
```
self.balances = [0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE, 0x5E8422345238F34275888049021821E8E08CAa1f];
```
`price_oracle()`  returns the 1st token price denominated in the 0th token.

## Tools used
Manual analysis.

## Recommended Mitigation Steps
```solidity
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
    uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
        10 ** 18
    );
    // This line should be fixed: / -> *
    return ((10 ** 18 * frxAmount) * IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
}
```