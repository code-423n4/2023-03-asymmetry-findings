# LOW FINDINGS 
##
### [L-1] UNUSED RECEIVE() functions

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

    97:  receive() external payable {}

[WstEth.sol#L97](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L97)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
 
    126: receive() external payable {}

[SfrxEth.sol#L126](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L126)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

   246: receive() external payable {}

[SafEth.sol#L246](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L246)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol

   244: receive() external payable {}

##

### [L-2] MISSING CHECKS FOR ADDRESS(0X0) WHEN ASSIGNING VALUES TO ADDRESS STATE VARIABLES

Owner address is not checked before calling _transferOwnership function 

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }

[WstEth.sol#L33-L36](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L33-L36)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol

   function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }

[SfrxEth.sol#L36-L39](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L36-L39)

##

### [L-3] Sanity/Threshold/Limit Checks

Devoid of sanity/threshold/limit checks, critical parameters can be configured to invalid values, causing a variety of issues and breaking expected interactions within/between contracts.

FILE : FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

   function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }

[WstEth.sol#L48-L50](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48-L50)

[WstEth.sol#L56-L67](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56-L67)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol

   function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }

[SfrxEth.sol#L51-L53](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53)

##

### [L-4] LOSS OF PRECISION DUE TO ROUNDING

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

   60: uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;

[WstEth.sol#L60](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L60)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol

   uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
            (10 ** 18 - maxSlippage)) / 10 ** 18;

[SfrxEth.sol#L74-L75](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75)

    return ((10 ** 18 * frxAmount) /
            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());

[SfrxEth.sol#L115-L116](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L115-L116)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

   81:  else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

[SafEth.sol#L81](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L81)

   88: uint256 ethAmount = (msg.value * weight) / totalWeight;

[SafEth.sol#L88](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L88)

   uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
                depositAmount
            ) * depositAmount) / 10 ** 18;

[SafEth.sol#L92-L94](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L92-L94)

   98: uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;

[SafEth.sol#L98](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L98)

     uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;

[SafEth.sol#L115-L116](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L115-L116)

   uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                totalWeight;

[SafEth.sol#L149-L150](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L149-L150)

##

### [L-5] A single point of failure

The onlyOwner role has a single point of failure and onlyOwner can use critical a few functions.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses

File: contracts/SafEth/SafEth.sol

138:     function rebalanceToWeights() external onlyOwner {

168:     ) external onlyOwner {

185:     ) external onlyOwner {

205:     ) external onlyOwner {

214:     function setMinAmount(uint256 _minAmount) external onlyOwner {

223:     function setMaxAmount(uint256 _maxAmount) external onlyOwner {

232:     function setPauseStaking(bool _pause) external onlyOwner {

241:     function setPauseUnstaking(bool _pause) external onlyOwner {
File: contracts/SafEth/derivatives/Reth.sol

58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

107:     function withdraw(uint256 amount) external onlyOwner {

156:     function deposit() external payable onlyOwner returns (uint256) {
File: contracts/SafEth/derivatives/SfrxEth.sol

51:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

60:     function withdraw(uint256 _amount) external onlyOwner {

94:     function deposit() external payable onlyOwner returns (uint256) {
File: contracts/SafEth/derivatives/WstEth.sol

48:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

56:     function withdraw(uint256 _amount) external onlyOwner {

73:     function deposit() external payable onlyOwner returns (uint256) {

# NON CRITICAL FINDINGS 

##

### [NC-1] Named imports can be used

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

   import "../../interfaces/IDerivative.sol";
   import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
   import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
   import "../../interfaces/curve/IStEthEthPool.sol";
   import "../../interfaces/lido/IWStETH.sol";

[WstEth.sol#L4-L8](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L4-L8)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol

   import "../../interfaces/IDerivative.sol";
   import "../../interfaces/frax/IsFrxEth.sol";
   import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
   import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
   import "../../interfaces/curve/IFrxEthEthPool.sol";
   import "../../interfaces/frax/IFrxETHMinter.sol";

[SfrxEth.sol#L4-L9](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L4-L9)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

      import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
      import "../interfaces/IWETH.sol";
      import "../interfaces/uniswap/ISwapRouter.sol";
      import "../interfaces/lido/IWStETH.sol";
      import "../interfaces/lido/IstETH.sol";
      import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
      import "./SafEthStorage.sol";
      import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

[SafEth.sol#L4-L11](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L4-L11)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol

    import "../../interfaces/IDerivative.sol";
    import "../../interfaces/frax/IsFrxEth.sol";
    import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
    import "../../interfaces/rocketpool/RocketStorageInterface.sol";
    import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
    import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
    import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
    import "../../interfaces/IWETH.sol";
    import "../../interfaces/uniswap/ISwapRouter.sol";
    import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
    import "../../interfaces/uniswap/IUniswapV3Factory.sol";
    import "../../interfaces/uniswap/IUniswapV3Pool.sol";

[Reth.sol#L4-L15](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L4-L15)

##

### [NC-2] Imports can be grouped together

Consider importing interfaces first, then OpenZeppelin imports 

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

   import "../../interfaces/IDerivative.sol";
   import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
   import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
   import "../../interfaces/curve/IStEthEthPool.sol";
   import "../../interfaces/lido/IWStETH.sol";

[WstEth.sol#L4-L8](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L4-L8)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol

   import "../../interfaces/IDerivative.sol";
   import "../../interfaces/frax/IsFrxEth.sol";
   import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
   import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
   import "../../interfaces/curve/IFrxEthEthPool.sol";
   import "../../interfaces/frax/IFrxETHMinter.sol";

[SfrxEth.sol#L4-L9](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L4-L9)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

      import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
      import "../interfaces/IWETH.sol";
      import "../interfaces/uniswap/ISwapRouter.sol";
      import "../interfaces/lido/IWStETH.sol";
      import "../interfaces/lido/IstETH.sol";
      import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
      import "./SafEthStorage.sol";
      import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

[SafEth.sol#L4-L11](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L4-L11)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol

    import "../../interfaces/IDerivative.sol";
    import "../../interfaces/frax/IsFrxEth.sol";
    import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
    import "../../interfaces/rocketpool/RocketStorageInterface.sol";
    import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
    import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
    import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
    import "../../interfaces/IWETH.sol";
    import "../../interfaces/uniswap/ISwapRouter.sol";
    import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
    import "../../interfaces/uniswap/IUniswapV3Factory.sol";
    import "../../interfaces/uniswap/IUniswapV3Pool.sol";

[Reth.sol#L4-L15](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L4-L15)
 
##

### [NC-3] AVOID HARDCODED VALUES 

It is not good practice to hardcode values, but if you are dealing with addresses much less, these can change between implementations, networks or projects, so it is convenient to remove these values from the source code

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

    address public constant WST_ETH =
        0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0;
    address public constant LIDO_CRV_POOL =
        0xDC24316b9AE028F1497c275EB9192a3Ea0f67022;
    address public constant STETH_TOKEN =
        0xae7ab96520DE3A18E5e111B5EaAb095312D7fE84

[WstEth.sol#L13-L18](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L13-L18)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
 
    address public constant SFRX_ETH_ADDRESS =
        0xac3E018457B222d93114458476f3E3416Abbe38F;
    address public constant FRX_ETH_ADDRESS =
        0x5E8422345238F34275888049021821E8E08CAa1f;
    address public constant FRX_ETH_CRV_POOL_ADDRESS =
        0xa1F8A6807c402E4A15ef4EBa36528A3FED24E577;
    address public constant FRX_ETH_MINTER_ADDRESS =
        0xbAFA44EFE7901E04E39Dad13167D089C559c1138;

[SfrxEth.sol#L14-L21](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L14-L21)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol

    address public constant ROCKET_STORAGE_ADDRESS =
        0x1d8f8f00cfa6758d7bE78336684788Fb0ee0Fa46;
    address public constant W_ETH_ADDRESS =
        0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;
    address public constant UNISWAP_ROUTER =
        0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45;
    address public constant UNI_V3_FACTORY =
        0x1F98431c8aD98523631AE4a59f267346ea31F984;

[Reth.sol#L20-L27](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L20-L27)

##

### [NC-4] Shorter the inheritance 

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

   contract SafEth is
    Initializable,
    ERC20Upgradeable,
    OwnableUpgradeable,
    SafEthStorage

[SafEth/SafEth.sol#L15-L19](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L15-L19)

##

### [NC-5] EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

    97:  receive() external payable {}

[WstEth.sol#L97](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L97)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
 
    126: receive() external payable {}

[SfrxEth.sol#L126](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L126)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

   246: receive() external payable {}

[SafEth.sol#L246](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L246)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol

   244: receive() external payable {}

##

### [NC-6] ADD PARAMETER TO EVENT-EMIT

Some event-emit description hasn’t parameter. Add to parameter for front-end website or client app , they can has that something has happened on the blockchain

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

  34: event Rebalanced();

[SafEth.sol#L34](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L34)

##

### [NC-7] Pragma float
All the contracts in scope are floating the pragma version.

Recommendation
Locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

Note that pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or a package.


  

   













NC-1	Return values of approve() not checked	3
NC-2	Event is missing indexed fields	5
NC-3	Constants should be defined rather than using magic numbers	1
NC-4	Functions not used internally could be marked external	8
L-1	Empty Function Body - Consider commenting why	4
L-2	Initializers could be front-run	9
L-3	Unsafe ERC20 operation(s)	3