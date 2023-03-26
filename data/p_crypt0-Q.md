# [Low/Informational] At least one derivative should be set in SafEth initialisation to avoid unexpected behaviours.

## Summary
`SafEth.sol`'s functions (i.e., 'stake') expect derivatives to be subscribed to `SafEth.sol`, however, this subscription of derivatives is not guaranteed at contract deployment.

For example:
```
        for (uint i = 0; i < derivativeCount; i++)
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                    derivatives[i].balance()) /
                10 ** 18;
```
[code](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71-L75)

## Implication
The full-extent of the implications of no derivatives being present in the initialisation of the contract `SafEth.sol` is not fully understood, hence the low/informational finding.

## Recommendations
There are a couple of measures the protocol could take.

The first of which, as discussed with dev-team lead @Toshi, is to include a derivative from initialisation `SafEth` intialisation, so that there is always at least one derivative present.

In order to accomplish this, the code could be altered as so:

```solidity
    /**
        @notice - Function to initialize values for the contracts
        @dev - This replaces the constructor for upgradeable contracts
        @param _tokenName - name of erc20
        @param _tokenSymbol - symbol of erc20
    */
    function initialize(
        string memory _tokenName,
        string memory _tokenSymbol
    ) external initializer {
        ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
        _transferOwnership(msg.sender);
        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum

        //@audit initialise a derivative (for example, WST_ETH which has already been deployed)
        address public constant WST_ETH = 0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0;
        uint256 EXAMPLE_WEIGHT = 400 // (weight as derived from docs)
        addDerivative(WST_ETH, EXAMPLE_WEIGHT);
    }
```
[code is here](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L42-L56)

[example weights are here](https://github.com/code-423n4/2023-03-asymmetry#additional-context)

***

The alternative recommendation, is to include a requirement that derivatives has some derivatives before running functions.

As such, you can alter the `stake` function thus:

```solidity
    /**
        @notice - Stake your ETH into safETH
        @dev - Deposits into each derivative based on its weight
        @dev - Mints safEth in a redeemable value which equals to the correct percentage of the total staked value
    */
    function stake() external payable {
        require(pauseStaking == false, "staking is paused");
        require(msg.value >= minAmount, "amount too low");
        require(msg.value <= maxAmount, "amount too high");

        // @audit add check that derivatives are subscribed.
        require(derivativeCount > 0, "no derivatives presently subscribed");

        uint256 underlyingValue = 0;


        // Getting underlying value in terms of ETH for each derivative
        for (uint i = 0; i < derivativeCount; i++)
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                    derivatives[i].balance()) /
                10 ** 18;


        uint256 totalSupply = totalSupply();
        uint256 preDepositPrice; // Price of safETH in regards to ETH
        if (totalSupply == 0)
            preDepositPrice = 10 ** 18; // initializes with a price of 1
        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;


        uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
        for (uint i = 0; i < derivativeCount; i++) {
            uint256 weight = weights[i];
            IDerivative derivative = derivatives[i];
            if (weight == 0) continue;
            uint256 ethAmount = (msg.value * weight) / totalWeight;


            // This is slightly less than ethAmount because slippage
            uint256 depositAmount = derivative.deposit{value: ethAmount}();
            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
                depositAmount
            ) * depositAmount) / 10 ** 18;
            totalStakeValueEth += derivativeReceivedEthValue;
        }
        // mintAmount represents a percentage of the total assets in the system
        uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
        _mint(msg.sender, mintAmount);
        emit Staked(msg.sender, msg.value, mintAmount);
    }
```

***