# QA report

## Author: rotcivegaf

## Non-critical

### [N‑01] Use scientific notation

[Look in the solidity documentation](https://docs.soliditylang.org/en/v0.8.17/types.html#rational-and-integer-literals)

```solidity
File: contracts/SafEth/derivatives/Reth.sol

/// @audit: `1 * 10 ** 16` to `0.01e18`
 44:        maxSlippage = (1 * 10 ** 16); // 1%

/// @audit: `10 ** 36` to `1e36`
171:            uint rethPerEth = (10 ** 36) / poolPrice();

/// @audit: `10 ** 18` to `1e18`
173:            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
174:                ((10 ** 18 - maxSlippage))) / 10 ** 18);
214:                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
215:        else return (poolPrice() * 10 ** 18) / (10 ** 18);
```

```solidity
File: contracts/SafEth/SafEth.sol

/// @audit: `5 * 10 ** 17` to `0.5e18`
54:        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum

/// @audit: `200 * 10 ** 18` to `200e18`
55:        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum

/// @audit: `10 ** 18` to `1e18`
75:                10 ** 18;
80:            preDepositPrice = 10 ** 18; // initializes with a price of 1
81:        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
94:            ) * depositAmount) / 10 ** 18;
98:        uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

/// @audit: `1 * 10 ** 16` to `0.01e18`
 38:        maxSlippage = (1 * 10 ** 16); // 1%

/// @audit: `10 ** 18` to `1e18`
 74:        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
 75:            (10 ** 18 - maxSlippage)) / 10 ** 18;
113:             10 ** 18
115:        return ((10 ** 18 * frxAmount) /
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

/// @audit: `1 * 10 ** 16` to `0.01e18`
35:        maxSlippage = (1 * 10 ** 16); // 1%

/// @audit: `10 ** 18` to `1e18`
60:        uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
87:        return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
```

### [N-02] Constants should be defined and documented rather than using magic numbers

```solidity
File: contracts/SafEth/derivatives/Reth.sol

171:            uint rethPerEth = (10 ** 36) / poolPrice();

173:            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *

174:                ((10 ** 18 - maxSlippage))) / 10 ** 18);

214:                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);

215:        else return (poolPrice() * 10 ** 18) / (10 ** 18);

241:        return (sqrtPriceX96 * (uint(sqrtPriceX96)) * (1e18)) >> (96 * 2);
```

```solidity
File: contracts/SafEth/SafEth.sol

75:                10 ** 18;

80:            preDepositPrice = 10 ** 18; // initializes with a price of 1

81:        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

94:            ) * depositAmount) / 10 ** 18;

98:        uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

 74:        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *

 75:            (10 ** 18 - maxSlippage)) / 10 ** 18;

113:             10 ** 18

115:        return ((10 ** 18 * frxAmount) /
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

60:        uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;

87:        return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
```

### [N‑03] Typo

```solidity
File: contracts/SafEth/SafEth.sol

/// @audit: `derivates` to `derivatives`
160:        @dev - If you want exact weights either do the math off chain or reset all existing derivates to the weights you want
```

### [N-04] Use `uint256` instead of `uint`

In the code the variables are defined as `uint256` and `uint`, use always `uint256`

```solidity
File: contracts/SafEth/derivatives/Reth.sol

171:            uint rethPerEth = (10 ** 36) / poolPrice();

241:        return (sqrtPriceX96 * (uint(sqrtPriceX96)) * (1e18)) >> (96 * 2);
```

```solidity
File: contracts/SafEth/SafEth.sol

 26:    event Staked(address indexed recipient, uint ethIn, uint safEthOut);

 27:    event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);

 28:    event WeightChange(uint indexed index, uint weight);

 31:        uint weight,

 32:        uint index

 71:        for (uint i = 0; i < derivativeCount; i++)

 84:        for (uint i = 0; i < derivativeCount; i++) {

 92:            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(

140:        for (uint i = 0; i < derivativeCount; i++) {

147:        for (uint i = 0; i < derivativeCount; i++) {

203:        uint _derivativeIndex,

204:        uint _slippage
```

### [N‑05] Named imports can be used

`import "<CONTRACT>.sol";` => `import {X} from "<CONTRACT>.sol";`

```solidity
File: contracts/SafEth/derivatives/Reth.sol

 4: import "../../interfaces/IDerivative.sol";
 5: import "../../interfaces/frax/IsFrxEth.sol";
 6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
 7: import "../../interfaces/rocketpool/RocketStorageInterface.sol";
 8: import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
 9: import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
10: import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
11: import "../../interfaces/IWETH.sol";
12: import "../../interfaces/uniswap/ISwapRouter.sol";
13: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
14: import "../../interfaces/uniswap/IUniswapV3Factory.sol";
15: import "../../interfaces/uniswap/IUniswapV3Pool.sol";
```

```solidity
File: contracts/SafEth/SafEth.sol

 4: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
 5: import "../interfaces/IWETH.sol";
 6: import "../interfaces/uniswap/ISwapRouter.sol";
 7: import "../interfaces/lido/IWStETH.sol";
 8: import "../interfaces/lido/IstETH.sol";
 9: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
10: import "./SafEthStorage.sol";
11: import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

4: import "../../interfaces/IDerivative.sol";
5: import "../../interfaces/frax/IsFrxEth.sol";
6: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
7: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
8: import "../../interfaces/curve/IFrxEthEthPool.sol";
9: import "../../interfaces/frax/IFrxETHMinter.sol";
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

4: import "../../interfaces/IDerivative.sol";
5: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
7: import "../../interfaces/curve/IStEthEthPool.sol";
8: import "../../interfaces/lido/IWStETH.sol";
```

### [N-06] Non-library/interface files should use fixed compiler versions, not floating ones

```solidity
File: contracts/SafEth/derivatives/Reth.sol

2: pragma solidity ^0.8.13;
```

```solidity
File: contracts/SafEth/SafEth.sol

2: pragma solidity ^0.8.13;
```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

2: pragma solidity ^0.8.13;
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

2: pragma solidity ^0.8.13;
```

### [N‑07] Unused imports

```solidity
File: contracts/SafEth/derivatives/Reth.sol

5: import "../../interfaces/frax/IsFrxEth.sol";

6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```

```solidity
File: contracts/SafEth/SafEth.sol

4: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

5: import "../interfaces/IWETH.sol";

6: import "../interfaces/uniswap/ISwapRouter.sol";

7: import "../interfaces/lido/IWStETH.sol";
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```

### [N‑08] Remove unused function parameter

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

/// @audit: Remove `_amount` parameter
111:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

/// @audit: Remove `_amount` parameter
86:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

### [N‑09] Missing event emitting in critical functions

```solidity
File: contracts/SafEth/derivatives/Reth.sol

/// @audit: emit new `maxSlippage`
42:    function initialize(address _owner) external initializer {
58:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

```solidity
File: contracts/SafEth/SafEth.sol

/// @audit: emit `ChangeMinAmount`
54:        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum

/// @audit: emit `ChangeMaxAmount`
55:        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

/// @audit: emit new `maxSlippage`
38:    function initialize(address _owner) external initializer {
51:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

/// @audit: emit new `maxSlippage`
33:    function initialize(address _owner) external initializer {
48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

### [N‑10] Lint

```solidity
File: contracts/SafEth/derivatives/Reth.sol

/// From:
91:        ISwapRouter.ExactInputSingleParams memory params = ISwapRouter
92:            .ExactInputSingleParams({
/// To:
        ISwapRouter.ExactInputSingleParams memory params =
            ISwapRouter.ExactInputSingleParams({
```

Wrong indentation:

```solidity
File: contracts/SafEth/derivatives/Reth.sol

/// @audit: emit new `maxSlippage`
124:                keccak256(
125:                    abi.encodePacked("contract.address", "rocketDepositPool")
126:                )
127:            );

129:                rocketDepositPoolAddress
130:            );

135:                keccak256(
136:                    abi.encodePacked(
137:                        "contract.address",
138:                        "rocketDAOProtocolSettingsDeposit"
139:                    )
140:                )
141:            );

143:                rocketProtocolSettingsAddress
144:            );

161:                keccak256(
162:                    abi.encodePacked("contract.address", "rocketDepositPool")
163:                )
164:            );

167:                rocketDepositPoolAddress
168:            );

190:                    keccak256(
191:                        abi.encodePacked("contract.address", "rocketTokenRETH")
192:                    )
193:                );

232:                keccak256(
233:                    abi.encodePacked("contract.address", "rocketTokenRETH")
234:                )
235:            );
```

Don't use extra parenthesis:

```solidity
File: contracts/SafEth/derivatives/Reth.sol

44:        maxSlippage = (1 * 10 ** 16); // 1%

173:            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
174:                ((10 ** 18 - maxSlippage))) / 10 ** 18);
```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

38:        maxSlippage = (1 * 10 ** 16); // 1%

115:        return ((10 ** 18 * frxAmount) /
116:            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

35:        maxSlippage = (1 * 10 ** 16); // 1%

80:        return (wstEthAmount);
```

### [N‑11] Use function `rethAddress`

```solidity
File: contracts/SafEth/derivatives/Reth.sol

121:        address rocketDepositPoolAddress = RocketStorageInterface(
122:            ROCKET_STORAGE_ADDRESS
123:        ).getAddress(
124:                keccak256(
125:                    abi.encodePacked("contract.address", "rocketDepositPool")
126:                )
127:            );

158:        address rocketDepositPoolAddress = RocketStorageInterface(
159:            ROCKET_STORAGE_ADDRESS
160:        ).getAddress(
161:                keccak256(
162:                    abi.encodePacked("contract.address", "rocketDepositPool")
163:                )
164:            );

187:            address rocketTokenRETHAddress = RocketStorageInterface(
188:                ROCKET_STORAGE_ADDRESS
189:            ).getAddress(
190:                    keccak256(
191:                        abi.encodePacked("contract.address", "rocketTokenRETH")
192:                    )
193:                );

229:        address rocketTokenRETHAddress = RocketStorageInterface(
230:            ROCKET_STORAGE_ADDRESS
231:        ).getAddress(
232:                keccak256(
233:                    abi.encodePacked("contract.address", "rocketTokenRETH")
234:                )
235:            );
```

## Low vulnerability

### [L‑01] If the `derivatives` and `weights` grow up enough, could get out of gas errors

The function `addDerivative` add element on `derivatives` and `weights` and grow the `derivativeCount`.
The `stake`, `unstake`, `rebalanceToWeights`, `adjustWeight` and `addDerivative` functions iterate over these mappings and may run out of gas during traversal
Recommended Mitigation Steps: setting a maximum `derivativeCount` in the `addDerivative` function

### [L‑02] The `maxAmount` should be greater or equal than `minAmount`

In function `setMaxAmount` of contract **SafEth** the maxAmount could be lower than `minAmount`, if this happened the `stake` function would be broken

Recommended Mitigation Steps: check if the maximum amount is greater or equal than minimum amount in the `setMaxAmount` function

### [L‑03] The `maxSlippage` should be lower than or equal `10 ** 18`

The function `setMaxSlippage` of the derivatives contracts could broke the `deposit` function if the `slippage` setted it's too high(greater than `10 ** 18`).
Broken the subtract operation of the derivatives contracts:
  - [`uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L60)
  - [`(10 ** 18 - maxSlippage)) / 10 ** 18;`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L75)
  - [`((10 ** 18 - maxSlippage))) / 10 ** 18);`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L174)

Recommended Mitigation Steps: check if the maximum slippage is lower or equal than `10 ** 18` in the `maxSlippage` function of the derivatives contracts

### [L‑04] Front-runnable initializers

Al contracts have a initialize function how can be front-runnable by a malicious actor

```solidity
File: contracts/SafEth/SafEth.sol

48:    function initialize(
49:        string memory _tokenName,
50:        string memory _tokenSymbol
51:    ) external initializer {
52:        ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
53:        _transferOwnership(msg.sender);
54:        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
55:        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
56:    }
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

33:    function initialize(address _owner) external initializer {
34:        _transferOwnership(_owner);
35:        maxSlippage = (1 * 10 ** 16); // 1%
36:    }
```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

36:    function initialize(address _owner) external initializer {
37:        _transferOwnership(_owner);
38:        maxSlippage = (1 * 10 ** 16); // 1%
39:    }
```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

42:    function initialize(address _owner) external initializer {
43:        _transferOwnership(_owner);
44:        maxSlippage = (1 * 10 ** 16); // 1%
45:    }
```

Recommended Mitigation Steps: Setting the owner in the contract's constructor to the msg.sender and adding the onlyOwner modifier to all initializers

### [L‑05] Stuck dust in **SafEth** contract for division

When `stake` in the contract `SafEth` some WEIs could be stuck in the contract because the equation [`uint256 ethAmount = (msg.value * weight) / totalWeight;`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L88), in example: 
ethAmount = (99 * 1) / 100 = 0.99 = 0 => lost 1 wei