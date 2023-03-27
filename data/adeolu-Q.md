## Unused argument in ethPerDerivative function in SfrxEth and WstEth contracts
 The ethPerDerivative function in SfrxEth and WstEth contracts accepts `uint256 _amount` argument which is unused in the logic of the function. It is not needed, it can be removed. 

- **Proof of Concept**
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L86
```
    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
    }
```

In this snippet from the ethPerDerivative function in WstEth.sol, it can be seen that `_amount` is unused. 

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L111

```
    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
            10 ** 18
        );
        return ((10 ** 18 * frxAmount) /
            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
    }
```
In this snippet from the ethPerDerivative function in SfrxEth.sol, it can be seen that `_amount` is unused. 

- **Tools Used**
  VS Code 
- **Recommended Mitigation Steps**
Remove the unused argument `_amount` from the code. 