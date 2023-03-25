### Using private rather than public for constants, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

There are 11 instances of this issue.

```
File: ./contracts/SafEth/derivatives/Reth.sol

20:     address public constant ROCKET_STORAGE_ADDRESS =
22:     address public constant W_ETH_ADDRESS =
24:     address public constant UNISWAP_ROUTER =
26:     address public constant UNI_V3_FACTORY =
```

```
File: ./contracts/SafEth/derivatives/SfrxEth.sol

14:     address public constant SFRX_ETH_ADDRESS =
16:     address public constant FRX_ETH_ADDRESS =
18:     address public constant FRX_ETH_CRV_POOL_ADDRESS =
20:     address public constant FRX_ETH_MINTER_ADDRESS =
```

```
File: ./contracts/SafEth/derivatives/WstEth.sol

13:     address public constant WST_ETH =
15:     address public constant LIDO_CRV_POOL =
17:     address public constant STETH_TOKEN =
```



