## Gas Optimization Findings
| Number |Issues Details |
|:--:|:-------|
|[GAS-01]| Use if statements instead of require and revert with custom errors instead of using strings
|[GAS-02]| Make some constants private
|[GAS-03]| Use calldata type instead of memory
|[GAS-04]| Compute off-chain and use directly the hashes of contract addresses if possible
|[GAS-05]| Streamline if/else statements to save gas
|[GAS-06]| Assign a local variable for for loops when comparing to state variables to save from SLOAD operations
***

## [GAS-01] Use if statements instead of require and revert with custom errors instead of using strings

Even thought it is a public finding I would strongly advise to replace all require statements if statements and use custom errors.
There are many places in the contract where require is being used and that can save quite a bit of gas.

SafEth.sol

require(pauseStaking == false, "staking is paused");
require(msg.value >= minAmount, "amount too low");
require(msg.value <= maxAmount, "amount too high");

require(pauseUnstaking == false, "unstaking is paused");

```
Reth.sol
require(sent, "Failed to send Ether");
require(rethBalance2 > rethBalance1, "No rETH was minted");

```

SfrxEth.sol
require(sent, "Failed to send Ether");

```

WstEth.sol
require(sent, "Failed to send Ether");

```
## |[GAS-02]| Make some constants private

Reth.sol
All of these can be made private to save gas

address public constant ROCKET_STORAGE_ADDRESS =
    0x1d8f8f00cfa6758d7bE78336684788Fb0ee0Fa46;
address public constant W_ETH_ADDRESS =
    0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;
address public constant UNISWAP_ROUTER =
    0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45;
address public constant UNI_V3_FACTORY =
    0x1F98431c8aD98523631AE4a59f267346ea31F984;

```
SfrxEth.sol

address public constant SFRX_ETH_ADDRESS =
    0xac3E018457B222d93114458476f3E3416Abbe38F;
address public constant FRX_ETH_ADDRESS =
    0x5E8422345238F34275888049021821E8E08CAa1f;
address public constant FRX_ETH_CRV_POOL_ADDRESS =
    0xa1F8A6807c402E4A15ef4EBa36528A3FED24E577;
address public constant FRX_ETH_MINTER_ADDRESS =
    0xbAFA44EFE7901E04E39Dad13167D089C559c1138;

```
WstEth.sol

address public constant WST_ETH =
    0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0;
address public constant LIDO_CRV_POOL =
    0xDC24316b9AE028F1497c275EB9192a3Ea0f67022;
address public constant STETH_TOKEN =
    0xae7ab96520DE3A18E5e111B5EaAb095312D7fE84;


## |[GAS-03]| Use calldata type instead of memory

SafEth.sol
Here we can use calldata type instead of memory
function initialize(
        string memory _tokenName,
        string memory _tokenSymbol
    )

## |[GAS-04]| Compute off-chain and use directly the hashes of contract addresses if possible
There are multiple places in the contract where we use keccak256 to hash some ABI of the contract. If this can be done off-chain and used in the contract - we will save quite a bit of gas.

```
Reth.sol
We can pass the already generated hash of contract.address and the second parameter. There are 6 occurrences of this in this contract.

RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
            keccak256(
                abi.encodePacked("contract.address", "rocketTokenRETH")
            )
        );


address rocketDepositPoolAddress = RocketStorageInterface(
        ROCKET_STORAGE_ADDRESS
    ).getAddress(
        keccak256(
            abi.encodePacked("contract.address", "rocketDepositPool")
        )
    );

address rocketProtocolSettingsAddress = RocketStorageInterface(
    ROCKET_STORAGE_ADDRESS
).getAddress(
        keccak256(
            abi.encodePacked(
                "contract.address",
                "rocketDAOProtocolSettingsDeposit"
                )
            )
        );

address rocketTokenRETHAddress = RocketStorageInterface(
            ROCKET_STORAGE_ADDRESS
        ).getAddress(
                keccak256(
                    abi.encodePacked("contract.address", "rocketTokenRETH")
                )
            );

address rocketTokenRETHAddress = RocketStorageInterface(
        ROCKET_STORAGE_ADDRESS
    ).getAddress(
            keccak256(
                abi.encodePacked("contract.address", "rocketTokenRETH")
            )
        );

```
## |[GAS-05]| Streamline if/else statements to save gas

```

Reth.sol
Change the order of the if check and check if amount is bigger or equal than the minimum deposit size of the pool. 
We will save calling the getBalance function and doing the addition and save gas.
BEFORE:
return
    rocketDepositPool.getBalance() + _amount <=
    rocketDAOProtocolSettingsDeposit.getMaximumDepositPoolSize() &&
    _amount >= rocketDAOProtocolSettingsDeposit.getMinimumDeposit();

AFTER:
_amount >= rocketDAOProtocolSettingsDeposit.getMinimumDeposit() &&
rocketDepositPool.getBalance() + _amount <=
    rocketDAOProtocolSettingsDeposit.getMaximumDepositPoolSize()

You can remove the else keyword here.
BEFORE:
if (poolCanDeposit(_amount))
    return RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
    else return (poolPrice() * 10 ** 18) / (10 ** 18);

AFTER:
if (poolCanDeposit(_amount))
    return RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
    return (poolPrice() * 10 ** 18) / (10 ** 18);

```

SafEth.sol
Move the if (weight == 0 ) check to be immediately after declaring the variable. You will save gas from sloading from the derivatives mapping (derivatives[i])
BEFORE:
for (uint i = 0; i < derivativeCount; i++) {
        uint256 weight = weights[i];
        IDerivative derivative = derivatives[i];
        if (weight == 0) continue;
        uint256 ethAmount = (msg.value * weight) / totalWeight;

AFTER:
for (uint i = 0; i < derivativeCount; i++) {
        uint256 weight = weights[i];
        if (weight == 0) continue;
        IDerivative derivative = derivatives[i];
        uint256 ethAmount = (msg.value * weight) / totalWeight;
```
```

## |[GAS-06]| Assign a local variable for for loops when comparing to state variables to save from SLOAD operations
Instead of reading directly from state variables and wasting gas on SLOAD operations every time we iterate over the loop, we can assign a local variable to be the length of what we are checking against and do an SLOAD operation just once.

SafEth.sol
BEFORE:
for (uint i = 0; i < derivativeCount; i++)
AFTER:
uint derivativeCountLength = derivativeCount
for (uint i = 0; i < derivativeCountLength; i++)

BEFORE:
for (uint i = 0; i < derivativeCount; i++) {
            if (derivatives[i].balance() > 0)
                derivatives[i].withdraw(derivatives[i].balance());
        }
AFTER:
uint derivativeCountLength = derivativeCount
for (uint i = 0; i < derivativeCountLength; i++) {
            if (derivatives[i].balance() > 0)
                derivatives[i].withdraw(derivatives[i].balance());
        }