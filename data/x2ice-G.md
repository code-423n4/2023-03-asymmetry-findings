 ## Using private rather than public for constants, saves gas

There are 11 instances of this issue:

```
File: contracts/SafEth/derivatives/Reth.sol

20    address public constant ROCKET_STORAGE_ADDRESS =
        0x1d8f8f00cfa6758d7bE78336684788Fb0ee0Fa46;

22    address public constant W_ETH_ADDRESS =
        0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;

24    address public constant UNISWAP_ROUTER =
        0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45;

26    address public constant UNI_V3_FACTORY =
        0x1F98431c8aD98523631AE4a59f267346ea31F984;
```

```
File: contracts/SafEth/derivatives/SfrxEth.sol

14    address public constant SFRX_ETH_ADDRESS =
        0xac3E018457B222d93114458476f3E3416Abbe38F;

16    address public constant FRX_ETH_ADDRESS =
        0x5E8422345238F34275888049021821E8E08CAa1f;

18    address public constant FRX_ETH_CRV_POOL_ADDRESS =
        0xa1F8A6807c402E4A15ef4EBa36528A3FED24E577;

20    address public constant FRX_ETH_MINTER_ADDRESS =
        0xbAFA44EFE7901E04E39Dad13167D089C559c1138;
```

```
File: contracts/SafEth/derivatives/WstEth.sol

13    address public constant WST_ETH =
        0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0;

15    address public constant LIDO_CRV_POOL =
        0xDC24316b9AE028F1497c275EB9192a3Ea0f67022;

17    address public constant STETH_TOKEN =
        0xae7ab96520DE3A18E5e111B5EaAb095312D7fE84;
```

## Using bools for storage incurs overhead

```
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27

Use uint256(1) and uint256(2) for true/false

```
File: contracts/SafEth/SafEthStorage.sol

16 bool public pauseStaking;

17 bool public pauseUnstaking;
```

## Using custom errors are consumes less gas then require with a string explanation 

For instance you can modify following function ```File: contracts/SafEth/SafEth.sol function stake() external payable``` in this way

```
error StakingIsPaused();
error AmountTooLow();
error AmountTooHight();

function stake() external payable {
    if (pauseStaking == false) revert StakingIsPaused();
    if (msg.value >= minAmount == false) revert AmountTooLow();
    if (msg.value <= maxAmount) revert AmountTooHight();
    ...
}
```