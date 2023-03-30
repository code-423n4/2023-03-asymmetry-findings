## Low and Non-Critical Issues Summary
| Number |Issues Details |
|:--:|:-------|
|[NC-01]| Use latest Solidity version
|[NC-02]| Use stable pragma statement
|[NC-03]| Use named imports instead of plain `import FILE.SOL`
|[NC-04]| Constants should be defined rather than using magic numbers
|[NC-05]| You can use named parameters in mapping types
|[L-01]| Missing events for critical parameter changes
|[L-02]| Possible division by 0 if contracts are configured improperly
|[L-03]| Possible to pass address 0 in functions that pass ownership

***

## [NC-01] Use latest Solidity version

Solidity pragma versioning should be upgraded to latest available version - 0.8.19

***

## |[NC-02]| Use stable pragma statement

Using a floating pragma statement `^0.8.13` is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode.

## |[NC-03]| Use named imports instead of plain `import FILE.SOL`

**Recommendation:**
`import {contract1, interface1} from "filename.sol";

## |[NC-04]| Constants should be defined rather than using magic numbers

SafEth.sol
minAmount = 5 * 10 ** 17; 
maxAmount = 200 * 10 ** 18;
preDepositPrice = 10 ** 18;
underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                    derivatives[i].balance()) /
                10 ** 18;
preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply
uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
                depositAmount
            ) * depositAmount) / 10 ** 18;

Reth.sol
Define "RocketPool" as a private constant and return it in the method
function name() public pure returns (string memory) {
        return "RocketPool";
    }
***
uint rethPerEth = (10 ** 36) / poolPrice();
uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
                ((10 ** 18 - maxSlippage))) / 10 ** 18);
(poolPrice() * 10 ** 18) / (10 ** 18);
factory.getPool(rocketTokenRETHAddress, W_ETH_ADDRESS, 500)
return (sqrtPriceX96 * (uint(sqrtPriceX96)) * (1e18)) >> (96 * 2);
***
WstEth.sol
Define "Lido" as a private constant and return it in the method
function name() public pure returns (string memory) {
        return "Lido";
}

return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
***

SfrxEth.sol

uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
            (10 ** 18 - maxSlippage)) / 10 ** 18;

Extract "Frax" to a private constant and return it in the method
function name() public pure returns (string memory) {
        return "Frax";
    }

## [NC-05] You can use named parameters in mapping types
Even though it is out of contest scope I would suggest it:

From Solidity [0.8.18](https://blog.soliditylang.org/2023/02/01/solidity-0.8.18-release-announcement/) you can use named parameters in mapping types. This will make the code much more readable. 
SafEthStorage.sol - for example you can use named parameters here.
mapping(uint256 => IDerivative) public derivatives
mapping(uint256 => uint256) public weights;



## [L-01] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

Emitting events allows monitoring activities with off-chain monitoring tools.
I would suggest creating custom events and emitting them on critical parameter changes. I am giving an example for a few files. 

```
Reth.sol
Create a custom event DepositSuccessful(indexed address depositor)
```
function deposit(
uint256 rethMinted = rethBalance2 - rethBalance1;
emit DepositSuccessful(rethAddress())
```
WstEth.sol
Create a custom event DepositSuccessful(indexed address depositor)
function deposit(
uint256 wstEthAmount = wstEthBalancePost - wstEthBalancePre;
emit DepositSuccessful(WST_ETH)
```

***

## [L-02] POSSIBLE DIVISION BY 0 IF CONTRACTS ARE CONFIGURED IMPROPERLY
Possible division by 0 can appear contracts if they are configured improperly and cause errors. This will prevent the contract from executing.
I'd recommend implementing a check for those and throwing a custom error where they are implemented, even if it is in the constructor.
I am providing some examples.
SafEth.sol
Total weight can be 0
uint256 ethAmount = (msg.value * weight) / totalWeight;
Total supply can be 0
_safEthAmount) / safEthTotalSupply;
```
Reth.sol
Looks like poolPrice can be 0 if configured improperly
if (!poolCanDeposit(msg.value)) {
            uint rethPerEth = (10 ** 36) / poolPrice();
```

## |[L-03]| Possible to pass address 0 in functions that pass ownership
There are functions that accept an address as a parameter and I recommend to add a check whether someone is passing address 0.
This is not critical but mistakes can happen.

Reth.sol
function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }
```
function swapExactInputSingleHop(
        address _tokenIn,
        address _tokenOut,
        uint24 _poolFee,
        uint256 _amountIn,
        uint256 _minOut
    ) private returns (uint256 amountOut) {
        IERC20(_tokenIn).approve(UNISWAP_ROUTER, _amountIn);
```

SafEth.sol

function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress);
```
SfrxEth.sol

function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }

```
