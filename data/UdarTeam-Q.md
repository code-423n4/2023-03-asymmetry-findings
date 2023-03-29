### Issues Template
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| O | Ordinary | Often found issues |

| Total Found Issues | 16 |
|:--:|:--:|

### Low Risk Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-01] | Owner can renounce ownership | 4 |
| [L-02] | Use two-step ownership trasnfer | 4 |
| [L-03] | Validate weight parameter | 2 |


| Total Low Risk Issues | 3 |
|:--:|:--:|

### Non-Critical Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | Create your own import names instead of using the regular ones | 31 |
| [N-02] | Remove unused imports | 5 |
| [N-03] | Use latest OpenZeppelin libraries | 2 |
| [N-04] | Explicitly mark uint type size | 21 |
| [N-05] | Insufficient code coverage | - |


| Total Non-Critical Issues | 5 |
|:--:|:--:|

### Refactor Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [R-01] | Fix wrong NatSpec comment | 1 |
| [R-02] | Always include curly brackets | 7 |
| [R-03] | Follow Solidity style guide order of functions | 4 |
| [R-04] | Non-external functions should have underscore prefix | 4 |
| [R-05] | Remove unnecessary round brackets | 2 |


| Total Refactor Issues | 5 |
|:--:|:--:|

### Ordinary Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [O-01] | Use a more recent pragma version | 4 |
| [O-02] | Locking pragma | 4 |
| [O-03] | Increase NatSpec comments | 19 |


| Total Ordinary Issues | 3 |
|:--:|:--:|

### [L-01] Owner can renounce ownership

All contracts inherits ```OwnableUpgradeable``` which has a public onlyOwner ```renounceOwnership``` function.

```solidity
contracts/SafEth/SafEth.sol

15:    contract SafEth is
16:        Initializable,
17:        ERC20Upgradeable,
18:        OwnableUpgradeable,
19:        SafEthStorage
20:    {
```

```solidity
contracts/SafEth/derivatives/Reth.sol

19:    contract Reth is IDerivative, Initializable, OwnableUpgradeable {
```

```solidity
contracts/SafEth/derivatives/SfrxEth.sol

13:    contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {
```

```solidity
contracts/SafEth/derivatives/WstEth.sol

12:    contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
```

Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner. Since the project has many onlyOwner functions, it will cause a critical issue.

### [L-02] Use two-step ownership trasnfer

All contracts inherits ```OwnableUpgradeable``` which has a public onlyOwner ```transferOwnership``` function.

```solidity
contracts/SafEth/SafEth.sol

15:    contract SafEth is
16:        Initializable,
17:        ERC20Upgradeable,
18:        OwnableUpgradeable,
19:        SafEthStorage
20:    {
```

```solidity
contracts/SafEth/derivatives/Reth.sol

19:    contract Reth is IDerivative, Initializable, OwnableUpgradeable {
```

```solidity
contracts/SafEth/derivatives/SfrxEth.sol

13:    contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {
```

```solidity
contracts/SafEth/derivatives/WstEth.sol

12:    contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
```

Use a two-step process to transfer the ownership which is more secure. Reference: [OpenZeppelin Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol).

### [L-03] Validate weight parameter

The contest's README says following: "Weights are not set in percentage out of 100, so if you set derivatives weights to 400, 400, and 200 they will be 40%, 40%, and 20% respectively."

The ```addDerivative``` and ```adjustWeight``` functions doesn't check for ```_weight``` parameter's value, which can be set to anything. Since the functions are guarded with onlyOwner modifier, the likelihood is low, but the impact can be high.

```solidity
contracts/SafEth/SafEth.sol

function adjustWeight(
    uint256 _derivativeIndex,
    uint256 _weight
) external onlyOwner {
    weights[_derivativeIndex] = _weight;
    uint256 localTotalWeight = 0;
    for (uint256 i = 0; i < derivativeCount; i++)
        localTotalWeight += weights[i];
    totalWeight = localTotalWeight;
    emit WeightChange(_derivativeIndex, _weight);
}

...

function addDerivative(
    address _contractAddress,
    uint256 _weight
) external onlyOwner {
    derivatives[derivativeCount] = IDerivative(_contractAddress);
    weights[derivativeCount] = _weight;
    derivativeCount++;

    uint256 localTotalWeight = 0;
    for (uint256 i = 0; i < derivativeCount; i++)
        localTotalWeight += weights[i];
    totalWeight = localTotalWeight;
    emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
}
```

### [N-01] Create your own import names instead of using the regular ones

For better readability, you should name the imports instead of using the regular ones.

```solidity
contracts/SafEth/SafEth.sol

4:    import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
5:    import "../interfaces/IWETH.sol";
6:    import "../interfaces/uniswap/ISwapRouter.sol";
7:    import "../interfaces/lido/IWStETH.sol";
8:    import "../interfaces/lido/IstETH.sol";
9:    import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
10:    import "./SafEthStorage.sol";
11:    import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
```

```solidity
contracts/SafEth/derivatives/Reth.sol

4:    import "../../interfaces/IDerivative.sol";
5:    import "../../interfaces/frax/IsFrxEth.sol";
6:    import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
7:    import "../../interfaces/rocketpool/RocketStorageInterface.sol";
8:    import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
9:    import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
10:    import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
11:    import "../../interfaces/IWETH.sol";
12:    import "../../interfaces/uniswap/ISwapRouter.sol";
13:    import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
14:    import "../../interfaces/uniswap/IUniswapV3Factory.sol";
15:    import "../../interfaces/uniswap/IUniswapV3Pool.sol";
```

```solidity
contracts/SafEth/derivatives/SfrxEth.sol

4:    import "../../interfaces/IDerivative.sol";
5:    import "../../interfaces/frax/IsFrxEth.sol";
6:    import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
7:    import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
8:    import "../../interfaces/curve/IFrxEthEthPool.sol";
9:    import "../../interfaces/frax/IFrxETHMinter.sol";
```

```solidity
contracts/SafEth/derivatives/WstEth.sol

4:    import "../../interfaces/IDerivative.sol";
5:    import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
6:    import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
7:    import "../../interfaces/curve/IStEthEthPool.sol";
8:    import "../../interfaces/lido/IWStETH.sol";
```

### [N-02] Remove unused imports

```solidity
contracts/SafEth/SafEth.sol

5:    import "../interfaces/IWETH.sol";
6:    import "../interfaces/uniswap/ISwapRouter.sol";
7:    import "../interfaces/lido/IWStETH.sol";
8:    import "../interfaces/lido/IstETH.sol";
```

```solidity
contracts/SafEth/derivatives/Reth.sol

5:    import "../../interfaces/frax/IsFrxEth.sol";
```

### [N-03] Use latest OpenZeppelin libraries

New versions of the OpenZeppelin libraries introduce critical bug fixes, optimizations and refactoring.

Current versions in `package.json`

```text
"@openzeppelin/contracts": "^4.8.0",
"@openzeppelin/contracts-upgradeable": "^4.8.1",
```

The most recent version is [v4.8.2](https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.8.2).

### [N-04] Explicitly mark uint type size

Explicitly marking uint type size improves code readability and consistency.

```solidity
contracts/SafEth/SafEth.sol    19 Instances
contracts/SafEth/derivatives/Reth.sol    2 Instances
```

### [N-05] Insufficient code coverage

Best practice is to come as close to 100% as possible for smart contracts.

According to documentation, the coverage is 92%.

### [R-01] Fix wrong NatSpec comment

```adjustWeight``` function in ```SafEth.sol``` has same @notice comment as ```addDerivative``` function.

```solidity
contracts/SafEth/SafEth.sol

158:    @notice - Adds new derivative to the index fund
178:    @notice - Adds new derivative to the index fund
```

### [R-02] Always include curly brackets

Always include curly brackets, even for one-line if statements or for loops. The reason for this is because omitting curly brackets increases the chance of writing buggy code.

```solidity
contracts/SafEth/SafEth.sol

71:    for (uint i = 0; i < derivativeCount; i++)
72:        underlyingValue += 
73:            (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74:                derivatives[i].balance()) /
75:            10 ** 18;

79:    if (totalSupply == 0)
80:        preDepositPrice = 10 ** 18; // initializes with a price of 1
81:    else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

141:    if (derivatives[i].balance() > 0)
142:        derivatives[i].withdraw(derivatives[i].balance());

148:    if (weights[i] == 0 || ethAmountToRebalance == 0) continue;

171:    for (uint256 i = 0; i < derivativeCount; i++)
172:        localTotalWeight += weights[i];

191:    for (uint256 i = 0; i < derivativeCount; i++)
192:        localTotalWeight += weights[i];
```

```solidity
contracts/SafEth/derivatives/Reth.sol

212:    if (poolCanDeposit(_amount))
213:        return
214:            RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
215:    else return (poolPrice() * 10 ** 18) / (10 ** 18);
```

### [R-03] Follow Solidity style guide order of functions

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered:

- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private

#### Instances

```solidity
contracts/SafEth/SafEth.sol
contracts/SafEth/derivatives/Reth.sol
contracts/SafEth/derivatives/SfrxEth.sol
contracts/SafEth/derivatives/WstEth.sol
```

### [R-04] Non-external functions should have underscore prefix

Leading underscores allow you to immediately recognize the intent of such functions, but more importantly, if you change a function from non-external to external (including public) and rename it accordingly, this forces you to review every call site while renaming.

```solidity
contracts/SafEth/derivatives/Reth.sol

66:    function rethAddress() private view returns (address) {
    
83:    function swapExactInputSingleHop(
84:        address _tokenIn,
85:        address _tokenOut,
86:        uint24 _poolFee,
87:        uint256 _amountIn,
88:        uint256 _minOut
89:    ) private returns (uint256 amountOut) {
    
120:    function poolCanDeposit(uint256 _amount) private view returns (bool) {

228:    function poolPrice() private view returns (uint256) {
```

### [R-05] Remove unnecessary round brackets

```solidity
contracts/SafEth/derivatives/Reth.sol

202:    return (rethMinted);
```

```solidity
contracts/SafEth/derivatives/WstEth.sol

80:    return (wstEthAmount);
```

### [O-01] Use a more recent pragma version

One of the most important best practices for writing secure Solidity code is to always use the latest version of the language. New versions of Solidity may include security updates and bug fixes that can help protect your code from potential vulnerabilities.

```solidity
contracts/SafEth/SafEth.sol

2:    pragma solidity ^0.8.13;
```

```solidity
contracts/SafEth/derivatives/Reth.sol

2:    pragma solidity ^0.8.13;
```

```solidity
contracts/SafEth/derivatives/SfrxEth.sol

2:    pragma solidity ^0.8.13;
```

```solidity
contracts/SafEth/derivatives/WstEth.sol

2:    pragma solidity ^0.8.13;
```

### [O-02] Locking pragma

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

```solidity
contracts/SafEth/SafEth.sol

2:    pragma solidity ^0.8.13;
```

```solidity
contracts/SafEth/derivatives/Reth.sol

2:    pragma solidity ^0.8.13;
```

```solidity
contracts/SafEth/derivatives/SfrxEth.sol

2:    pragma solidity ^0.8.13;
```

```solidity
contracts/SafEth/derivatives/WstEth.sol

2:    pragma solidity ^0.8.13;
```

### [O-03] Increase NatSpec comments

Some of the functions are missing NatSpec @param and @return tags. Properly commenting the code is very important because it makes it easier for developers and other users to understand the code. Another reason for properly commenting the smart contracts is to make it easier for auditing companies to audit the code.

```solidity
contracts/SafEth/derivatives/Reth.sol

66:    function rethAddress() private view returns (address) {

83:    function swapExactInputSingleHop(
84:        address _tokenIn,
85:        address _tokenOut,
86:        uint24 _poolFee,
87:        uint256 _amountIn,
88:        uint256 _minOut
89:    ) private returns (uint256 amountOut) {
    
107:    function withdraw(uint256 amount) external onlyOwner {

120:    function poolCanDeposit(uint256 _amount) private view returns (bool) {

156:    function deposit() external payable onlyOwner returns (uint256) {
    
211:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {

221:    function balance() public view returns (uint256) {
    
228:    function poolPrice() private view returns (uint256) {
```

```solidity
contracts/SafEth/derivatives/SfrxEth.sol

44:    function name() public pure returns (string memory) {

51:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
    
94:    function deposit() external payable onlyOwner returns (uint256) {

111:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
    
122:    function balance() public view returns (uint256) {
```

```solidity
contracts/SafEth/derivatives/WstEth.sol

41:    function name() public pure returns (string memory) {
    
48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
    
56:    function withdraw(uint256 _amount) external onlyOwner {
    
73:    function deposit() external payable onlyOwner returns (uint256) {
    
86:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
    
93:    function balance() public view returns (uint256) {
```
