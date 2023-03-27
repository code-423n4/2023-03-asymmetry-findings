**1.Event is missing indexed fields**
Indexing event fields can improve the accessibility of fields to off-chain tools that parse events. However, each index field adds extra gas cost during emission, so it's not always advisable to index the maximum of three fields per event. It's recommended to index three fields if there are three or more fields and gas usage is not a concern. For events with fewer than three fields, it's best to index all of them.

Therefore, all the event parameters in SafEth contract can be made indexed.

    event ChangeMinAmount(uint256 indexed minAmount);
    event ChangeMaxAmount(uint256 indexed maxAmount);
    event StakingPaused(bool indexed paused);
    event UnstakingPaused(bool indexed paused);
    event SetMaxSlippage(uint256 indexed index, uint256 slippage);
    event Staked(address indexed recipient, uint ethIn, uint safEthOut);
    event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);
    event WeightChange(uint indexed index, uint weight);
    event DerivativeAdded(
        address indexed contractAddress,
        uint weight,
        uint index
    );