Reth.sol :

QA-1

`deposit()` would revert for rocket derivative due to hard coded pool fee (500 is set)

        if (!poolCanDeposit(msg.value)) {
            uint rethPerEth = (10 ** 36) / poolPrice();


            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
                ((10 ** 18 - maxSlippage))) / 10 ** 18);


            IWETH(W_ETH_ADDRESS).deposit{value: msg.value}();
            uint256 amountSwapped = swapExactInputSingleHop(
                W_ETH_ADDRESS,
                rethAddress(),
                500, --------------------------------> audit observation - hardcoded fee value.
                msg.value,
                minOut
            );

When fee values are increased, one of the derivative would revert. And due to this, all staking would revert.


QA-2

Use storge gap since the contracts are upgradable.

There are more chances for storage slot Collison when new variable is added at the end of the `SafEthStorage`

Lets look at the SafEthStorage contract,

    contract SafEthStorage {
         bool public pauseStaking; // true if staking is paused
         bool public pauseUnstaking; // true if unstaking is pause
         uint256 public derivativeCount; // amount of derivatives added to contract
         uint256 public totalWeight; // total weight of all derivatives (used to calculate percentage of 
          derivative)
        uint256 public minAmount; // minimum amount to stake
        uint256 public maxAmount; // maximum amount to stake
        mapping(uint256 => IDerivative) public derivatives; // derivatives in the system
        mapping(uint256 => uint256) public weights; // weights for each derivative

The parameter `maxAmount` would be last one and any new variable can be added next to this maxAmount.

Lets look one of the contracts, `WstEth.sol`

    contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
    address public constant WST_ETH =
        0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0;
    address public constant LIDO_CRV_POOL =
        0xDC24316b9AE028F1497c275EB9192a3Ea0f67022;
    address public constant STETH_TOKEN =
        0xae7ab96520DE3A18E5e111B5EaAb095312D7fE84;


    uint256 public maxSlippage;


    // As recommended by https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable
    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }

First three variables are constant so they will not occupy any storage slot.
but the maxSlippage would be assgined in storage slot since it is a state variable.

Any new value added in SafEthStorage would affect the maxSlippage value.

Recommendation : use storage gap. Follow Open zeppalin recommendation




