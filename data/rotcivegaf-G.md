# Gas report

## Author: rotcivegaf

## [G-01] Add `unchecked{}` in operations where the operands cannot over/underflow

```solidity
File: contracts/SafEth/derivatives/Reth.sol

171:            uint rethPerEth = (10 ** 36) / poolPrice();

173:            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
174:                ((10 ** 18 - maxSlippage))) / 10 ** 18);

/// @audit: checked in L200: `require(rethBalance2 > rethBalance1, "No rETH was minted");`
201:            uint256 rethMinted = rethBalance2 - rethBalance1;

215:        else return (poolPrice() * 10 ** 18) / (10 ** 18);
```

```solidity
File: contracts/SafEth/SafEth.sol

 81:        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

 88:            uint256 ethAmount = (msg.value * weight) / totalWeight;

 94:            ) * depositAmount) / 10 ** 18;

 95:            totalStakeValueEth += derivativeReceivedEthValue;

 98:        uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;

116:                _safEthAmount) / safEthTotalSupply;

149:            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
150:               totalWeight;

188:        derivativeCount++;
```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

115:        return ((10 ** 18 * frxAmount) /
116:            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
```

## [G-02] `<x>++` costs more gas than `<x> = <x> + 1` for state variables

```solidity
File: contracts/SafEth/SafEth.sol

188:        derivativeCount++;
```

## [G-03] `constructor` could be marked as `payable`

```solidity
File: contracts/SafEth/derivatives/Reth.sol

33:    constructor() {
```

```solidity
File: contracts/SafEth/SafEth.sol

38:    constructor() {
```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

27:    constructor() {
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

24:    constructor() {
```

## [G-04] `onlyOwner` functions could be marked as `payable`

```solidity
File: contracts/SafEth/derivatives/Reth.sol

 58:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

107:    function withdraw(uint256 amount) external onlyOwner {
```

```solidity
File: contracts/SafEth/SafEth.sol

138:    function rebalanceToWeights() external onlyOwner {

168:    ) external onlyOwner {

185:    ) external onlyOwner {

205:    ) external onlyOwner {

214:    function setMinAmount(uint256 _minAmount) external onlyOwner {

223:    function setMaxAmount(uint256 _maxAmount) external onlyOwner {

232:    function setPauseStaking(bool _pause) external onlyOwner {

241:    function setPauseUnstaking(bool _pause) external onlyOwner {
```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

51:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

60:    function withdraw(uint256 _amount) external onlyOwner {
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

56:   function withdraw(uint256 _amount) external onlyOwner {
```

## [G-05] Use function saved var instead of `SLOAD`

```solidity
File: contracts/SafEth/SafEth.sol

/// @audit: `_minAmount` instead of `minAmount`
216:        emit ChangeMinAmount(minAmount);

/// @audit: `_maxAmount` instead of `maxAmount`
225:        emit ChangeMaxAmount(maxAmount);

/// @audit: `_pause` instead of `pauseStaking`
234:        emit StakingPaused(pauseStaking);

/// @audit: `_pause` instead of `pauseUnstaking`
243:        emit UnstakingPaused(pauseUnstaking);
```

## [G-06] Don't save one used value

```solidity
File: contracts/SafEth/derivatives/Reth.sol

/// From:
 91:        ISwapRouter.ExactInputSingleParams memory params = ISwapRouter
 92:            .ExactInputSingleParams({
 93:                tokenIn: _tokenIn,
 94:                tokenOut: _tokenOut,
 95:                fee: _poolFee,
 96:                recipient: address(this),
 97:                amountIn: _amountIn,
 98:                amountOutMinimum: _minOut,
 99:                sqrtPriceLimitX96: 0
100:            });
101:        amountOut = ISwapRouter(UNISWAP_ROUTER).exactInputSingle(params);
/// To:
        amountOut = ISwapRouter(UNISWAP_ROUTER).exactInputSingle(
            ISwapRouter.ExactInputSingleParams({
                tokenIn: _tokenIn,
                tokenOut: _tokenOut,
                fee: _poolFee,
                recipient: address(this),
                amountIn: _amountIn,
                amountOutMinimum: _minOut,
                sqrtPriceLimitX96: 0
            })
        );

/// From:
121:        address rocketDepositPoolAddress = RocketStorageInterface(
122:            ROCKET_STORAGE_ADDRESS
123:        ).getAddress(
124:                keccak256(
125:                    abi.encodePacked("contract.address", "rocketDepositPool")
126:                )
127:            );
128:        RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(
129:                rocketDepositPoolAddress
130:            );
/// To:
        RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(rethAddress());

/// From:
132:        address rocketProtocolSettingsAddress = RocketStorageInterface(
133:            ROCKET_STORAGE_ADDRESS
134:        ).getAddress(
135:                keccak256(
136:                    abi.encodePacked(
137:                        "contract.address",
138:                        "rocketDAOProtocolSettingsDeposit"
139:                    )
140:                )
141:            );
142:        RocketDAOProtocolSettingsDepositInterface rocketDAOProtocolSettingsDeposit = RocketDAOProtocolSettingsDepositInterface(
143:                rocketProtocolSettingsAddress
144:            );
/// To:
        RocketDAOProtocolSettingsDepositInterface rocketDAOProtocolSettingsDeposit = RocketDAOProtocolSettingsDepositInterface(
            RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
                keccak256(abi.encodePacked("contract.address", "rocketDAOProtocolSettingsDeposit"))
            )
        );

/// From:
158:        address rocketDepositPoolAddress = RocketStorageInterface(
159:            ROCKET_STORAGE_ADDRESS
160:        ).getAddress(
161:                keccak256(
162:                    abi.encodePacked("contract.address", "rocketDepositPool")
163:                )
164:            );
165:
166:        RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(
167:                rocketDepositPoolAddress
168:            );
169:
170:        if (!poolCanDeposit(msg.value)) {
171:            uint rethPerEth = (10 ** 36) / poolPrice();
172:
173:            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
174:                ((10 ** 18 - maxSlippage))) / 10 ** 18);
175:
176:            IWETH(W_ETH_ADDRESS).deposit{value: msg.value}();
177:            uint256 amountSwapped = swapExactInputSingleHop(
178:                W_ETH_ADDRESS,
179:                rethAddress(),
180:                500,
181:                msg.value,
182:                minOut
183:            );
184:
185:            return amountSwapped;
186:        } else {
187:            address rocketTokenRETHAddress = RocketStorageInterface(
188:                ROCKET_STORAGE_ADDRESS
189:            ).getAddress(
190:                    keccak256(
191:                        abi.encodePacked("contract.address", "rocketTokenRETH")
192:                    )
193:                );
194:            RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(
195:                rocketTokenRETHAddress
196:            );
197:            uint256 rethBalance1 = rocketTokenRETH.balanceOf(address(this));
198:            rocketDepositPool.deposit{value: msg.value}();
/// To:
        if (!poolCanDeposit(msg.value)) {
            uint rethPerEth = (10 ** 36) / poolPrice();

            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
                ((10 ** 18 - maxSlippage))) / 10 ** 18);

            IWETH(W_ETH_ADDRESS).deposit{value: msg.value}();
            uint256 amountSwapped = swapExactInputSingleHop(
                W_ETH_ADDRESS,
                rethAddress(),
                500,
                msg.value,
                minOut
            );

            return amountSwapped;
        } else {
            address rocketTokenRETHAddress = RocketStorageInterface(
                ROCKET_STORAGE_ADDRESS
            ).getAddress(
                    keccak256(
                        abi.encodePacked("contract.address", "rocketTokenRETH")
                    )
                );
            RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(
                rocketTokenRETHAddress
            );
            uint256 rethBalance1 = rocketTokenRETH.balanceOf(address(this));
            RocketDepositPoolInterface(rethAddress()).deposit{value: msg.value}();

/// From:
171:            uint rethPerEth = (10 ** 36) / poolPrice();
172:
173:            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
174:                ((10 ** 18 - maxSlippage))) / 10 ** 18);
/// To:
            uint256 minOut = (((((10 ** 36) / poolPrice() * msg.value) / 10 ** 18) *
                ((10 ** 18 - maxSlippage))) / 10 ** 18);

/// From:
177:            uint256 amountSwapped = swapExactInputSingleHop(
178:                W_ETH_ADDRESS,
179:                rethAddress(),
180:                500,
181:                msg.value,
182:                minOut
183:            );
184:
185:            return amountSwapped;
/// To:
            return swapExactInputSingleHop(
                W_ETH_ADDRESS,
                rethAddress(),
                500,
                msg.value,
                minOut
            );

/// From:
187:            address rocketTokenRETHAddress = RocketStorageInterface(
188:                ROCKET_STORAGE_ADDRESS
189:            ).getAddress(
190:                    keccak256(
191:                        abi.encodePacked("contract.address", "rocketTokenRETH")
192:                    )
193:                );
194:            RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(
195:                rocketTokenRETHAddress
196:            );
/// To:
            RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(rethAddress());

/// From:
201:            uint256 rethMinted = rethBalance2 - rethBalance1;
202:            return (rethMinted);
/// To:
            return rethBalance2 - rethBalance1;

/// From:
229:        address rocketTokenRETHAddress = RocketStorageInterface(
230:            ROCKET_STORAGE_ADDRESS
231:        ).getAddress(
232:                keccak256(
233:                    abi.encodePacked("contract.address", "rocketTokenRETH")
234:                )
235:            );
236:        IUniswapV3Factory factory = IUniswapV3Factory(UNI_V3_FACTORY);
237:        IUniswapV3Pool pool = IUniswapV3Pool(
238:            factory.getPool(rocketTokenRETHAddress, W_ETH_ADDRESS, 500)
239:        );
240:        (uint160 sqrtPriceX96, , , , , , ) = pool.slot0();
/// To:
        (uint160 sqrtPriceX96, , , , , , ) = IUniswapV3Pool(
            IUniswapV3Factory(UNI_V3_FACTORY).getPool(rethAddress(), W_ETH_ADDRESS, 500)
        ).slot0();

```

```solidity
File: contracts/SafEth/SafEth.sol

/// From:
88:            uint256 ethAmount = (msg.value * weight) / totalWeight;
91:            uint256 depositAmount = derivative.deposit{value: ethAmount}();
/// To:
            uint256 depositAmount = derivative.deposit{value: (msg.value * weight) / totalWeight}();

/// From:
92:            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
93:                depositAmount
94:            ) * depositAmount) / 10 ** 18;
95:            totalStakeValueEth += derivativeReceivedEthValue;
/// To:
            totalStakeValueEth += (derivative.ethPerDerivative(
                depositAmount
            ) * depositAmount) / 10 ** 18;

/// From:
121:        uint256 ethAmountAfter = address(this).balance;
122:        uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;
/// To:
        uint256 ethAmountToWithdraw = address(this).balance - ethAmountBefore;

/// From:
144:        uint256 ethAmountAfter = address(this).balance;
145:        uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
/// To:
        uint256 ethAmountToRebalance = address(this).balance - ethAmountBefore;
```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

/// From:
 74:        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
 75:            (10 ** 18 - maxSlippage)) / 10 ** 18;
 76:
 77:        IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).exchange(
 78:            1,
 79:            0,
 80:            frxEthBalance,
 81:            minOut
 82:        );
/// To:
        IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).exchange(
            1,
            0,
            frxEthBalance,
            (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
                (10 ** 18 - maxSlippage)) / 10 ** 18
        );

/// From:
 95:        IFrxETHMinter frxETHMinterContract = IFrxETHMinter(
 96:            FRX_ETH_MINTER_ADDRESS
 97:        );
101:        frxETHMinterContract.submitAndDeposit{value: msg.value}(address(this));
/// To:
        IFrxETHMinter(FRX_ETH_MINTER_ADDRESS).submitAndDeposit{value: msg.value}(address(this));

/// From:
102:        uint256 sfrxBalancePost = IERC20(SFRX_ETH_ADDRESS).balanceOf(
103:            address(this)
104:        );
105:        return sfrxBalancePost - sfrxBalancePre;
/// To:
        return IERC20(SFRX_ETH_ADDRESS).balanceOf(address(this)) - sfrxBalancePre;

/// From:
112:        uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
113:            10 ** 18
114:        );
115:        return ((10 ** 18 * frxAmount) /
116:            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
/// To:
        return ((10 ** 18 * IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(10 ** 18)) /
            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

/// From:
60;        uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
61;        IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);
/// To:
        IStEthEthPool(LIDO_CRV_POOL).exchange(
            1,
            0,
            stEthBal,
            (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18
        );

/// From:
78:        uint256 wstEthBalancePost = IWStETH(WST_ETH).balanceOf(address(this));
79:        uint256 wstEthAmount = wstEthBalancePost - wstEthBalancePre;
80:        return (wstEthAmount);
/// To:
        return IWStETH(WST_ETH).balanceOf(address(this)) - wstEthBalancePre;
```

## [G-07] Sload `derivatives` after the `continue`

```solidity
File: contracts/SafEth/SafEth.sol

/// From:
86:            IDerivative derivative = derivatives[i];
87:            if (weight == 0) continue;
/// To:
            if (weight == 0) continue;
            IDerivative derivative = derivatives[i];
```

## [G-08] Store in local var to save calls

```solidity
File: contracts/SafEth/SafEth.sol

/// @audit: store the `derivativeCount` in a local var before the for loops and use it in the for loops
71:        for (uint i = 0; i < derivativeCount; i++)
84:        for (uint i = 0; i < derivativeCount; i++) {

/// @audit: store the `totalWeight` in a local var before the for loop and use it in the for loop
88:            uint256 ethAmount = (msg.value * weight) / totalWeight;

/// @audit: store the `derivativeCount` in a local var before the for loop and use it in the for loop
113:        for (uint256 i = 0; i < derivativeCount; i++) {

/// @audit: store the `derivativeCount` in a local var before the for loops and use it in the for loops
140:        for (uint i = 0; i < derivativeCount; i++) {
147:        for (uint i = 0; i < derivativeCount; i++) {

/// From:
141:            if (derivatives[i].balance() > 0)
142:                derivatives[i].withdraw(derivatives[i].balance());
/// To:
        for (uint i = 0; i < derivativeCount; i++) {
            IDerivative derivative = derivatives[i];
            uint256 derBal = derivative.balance();
            if (derBal > 0)
                derivative.withdraw(derBal);
        }

/// From:
148:            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
149:            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
/// To:
            uint256 weight = weights[i];
            if (weight == 0 || ethAmountToRebalance == 0) continue;
            uint256 ethAmount = (ethAmountToRebalance * weight) /

/// @audit: store the `totalWeight` in a local var before the for loop and use it in the for loop
150:                totalWeight;

/// @audit: store the `derivativeCount` in a local var before the for loop and use it in the for loop
171:        for (uint256 i = 0; i < derivativeCount; i++)

/// From:
186:        derivatives[derivativeCount] = IDerivative(_contractAddress);
187:        weights[derivativeCount] = _weight;
188:        derivativeCount++;
189:
190:        uint256 localTotalWeight = 0;
191:        for (uint256 i = 0; i < derivativeCount; i++)
192:            localTotalWeight += weights[i];
193:        totalWeight = localTotalWeight;
194:        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
/// To:
        uint256 _derivativeCount = derivativeCount;
        derivatives[_derivativeCount] = IDerivative(_contractAddress);
        weights[_derivativeCount] = _weight;
        _derivativeCount = ++derivativeCount;

        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < _derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit DerivativeAdded(_contractAddress, _weight, _derivativeCount);
```

## [G-09] Calculate `underlyingValue` only if will used

```diff
@@ -65,20 +65,22 @@ contract SafEth is
         require(msg.value >= minAmount, "amount too low");
         require(msg.value <= maxAmount, "amount too high");

-        uint256 underlyingValue = 0;
-
-        // Getting underlying value in terms of ETH for each derivative
-        for (uint i = 0; i < derivativeCount; i++)
-            underlyingValue +=
-                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
-                    derivatives[i].balance()) /
-                10 ** 18;
-
         uint256 totalSupply = totalSupply();
         uint256 preDepositPrice; // Price of safETH in regards to ETH
         if (totalSupply == 0)
             preDepositPrice = 10 ** 18; // initializes with a price of 1
-        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
+        else {
+            uint256 underlyingValue = 0;
+
+            // Getting underlying value in terms of ETH for each derivative
+            for (uint i = 0; i < derivativeCount; i++)
+                underlyingValue +=
+                    (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
+                        derivatives[i].balance()) /
+                    10 ** 18;
+
+            preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
+        }
```

## [G-10] Join the mappings `derivatives` and `weights`

Using a struct can join the `derivatives` and the `weights` in the same mapping, using one slot of bytes32 per element

```solidity
struct Derivative {
    IDerivative addr:
    uint96 weight;
}

mapping(uint256 => Derivative) public derivatives;
```

## [G-11] Revert if don't have amount to rebalance

```diff
@@ -18,6 +18,8 @@ contract SafEth is
     OwnableUpgradeable,
     SafEthStorage
 {
+    error NoEthAmountToRebalance();
+
     event ChangeMinAmount(uint256 indexed minAmount);
     event ChangeMaxAmount(uint256 indexed maxAmount);
     event StakingPaused(bool indexed paused);
@@ -143,9 +145,10 @@ contract SafEth is
         }
         uint256 ethAmountAfter = address(this).balance;
         uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
+        if (ethAmountToRebalance == 0) revert NoEthAmountToRebalance();

         for (uint i = 0; i < derivativeCount; i++) {
-            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
+            if (weights[i] == 0) continue;
             uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                 totalWeight;
             // Price will change due to slippage
```
