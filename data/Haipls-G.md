## Derivative never deleting
Even when the weight for the `derivative` was set to 0, it still affects the price of transactions
*Instances:*
```
contracts\SafEth\derivatives\SafEth.sol

71:         for (uint i = 0; i < derivativeCount; i++)
72:             underlyingValue +=
73:                 (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74:                     derivatives[i].balance()) /
75:                 10 ** 18;

84:         for (uint i = 0; i < derivativeCount; i++) {
85:             uint256 weight = weights[i];
86:             IDerivative derivative = derivatives[i];
87:             if (weight == 0) continue;

113:         for (uint256 i = 0; i < derivativeCount; i++) {
114:             // withdraw a percentage of each asset based on the amount of safETH
115:             uint256 derivativeAmount = (derivatives[i].balance() *
116:                 _safEthAmount) / safEthTotalSupply;
117:             if (derivativeAmount == 0) continue; // if derivative empty ignore
118:             derivatives[i].withdraw(derivativeAmount);

```

## Duplicate code should be extracted into a separate internal function.
In the contracts, there are duplicates of identical pieces of code that should be extracted into internal functions for optimizing contract size, gas usage, and readability.

*Instances:*
```
contracts\SafEth\derivatives\Reth.sol

121:         address rocketDepositPoolAddress = RocketStorageInterface(
122:             ROCKET_STORAGE_ADDRESS
123:         ).getAddress(
124:                 keccak256(
125:                     abi.encodePacked("contract.address", "rocketDepositPool")
126:                 )
127:             );
128:        RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(
129:                rocketDepositPoolAddress
130:            );

158:         address rocketDepositPoolAddress = RocketStorageInterface(
159:             ROCKET_STORAGE_ADDRESS
160:         ).getAddress(
161:                 keccak256(
162:                     abi.encodePacked("contract.address", "rocketDepositPool")
163:                 )
164:             );
165: 
166:        RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(
167:                rocketDepositPoolAddress
168:            );
```


## `keccak256` for `abi.encodePacked("contract.address", ***)` should be pre calculate
The value of `keccak256` for constants string is immutable and can be written as a constant or directly in code, without having to be called on every call

*Instances:*
```solidity
contracts\SafEth\derivatives\Reth.sol

69:                 keccak256(
70:                     abi.encodePacked("contract.address", "rocketTokenRETH")
71:                 )

124:                 keccak256(
125:                     abi.encodePacked("contract.address", "rocketDepositPool")
126:                 )

135:                 keccak256(
136:                     abi.encodePacked(
137:                         "contract.address",
138:                         "rocketDAOProtocolSettingsDeposit"
139:                     )
140:                 )

161:                 keccak256(
162:                     abi.encodePacked("contract.address", "rocketDepositPool")
163:                 )

190:                     keccak256(
191:                         abi.encodePacked("contract.address", "rocketTokenRETH")
192:                     )

232:                 keccak256(
233:                     abi.encodePacked("contract.address", "rocketTokenRETH")
234:                 )

```


## Instead of call keccak256 from `contract.address, rocketTokenRETH` should be use private `rethAddress` method
In the `Reth.sol` contract, there are two instances of identical code that corresponds to the code in the `rethAddress()` method. It is recommended to use the `rethAddress()` call in those places instead.

```solidity
contracts\SafEth\derivatives\Reth.sol

66:     function rethAddress() private view returns (address) {
67:         return
68:             RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
69:                 keccak256(
70:                     abi.encodePacked("contract.address", "rocketTokenRETH")
71:                 )
72:             );
73:     }

187:             address rocketTokenRETHAddress = RocketStorageInterface(
188:                 ROCKET_STORAGE_ADDRESS
189:             ).getAddress(
190:                     keccak256(
191:                         abi.encodePacked("contract.address", "rocketTokenRETH")
192:                     )
193:                 );

229:         address rocketTokenRETHAddress = RocketStorageInterface(
230:             ROCKET_STORAGE_ADDRESS
231:         ).getAddress(
232:                 keccak256(
233:                     abi.encodePacked("contract.address", "rocketTokenRETH")
234:                 )
235:             );


```


## Redundant calls to storage
In the following cases, the call to storage or call the same method more than once to the same variable. Which is redundant and needs caching optimizations

### Method - `stake()`:
- variable `derivativeCount` 2 time in method
- call `derivatives[i].balance()` 2 time in loop
- call `derivatives[i]` 3 time in loop

```solidity
contracts\SafEth\derivatives\SafEth.sol

71:        for (uint i = 0; i < derivativeCount; i++)
84:        for (uint i = 0; i < derivativeCount; i++)

72:            underlyingValue +=
73:                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74:                    derivatives[i].balance()) /
75:                10 ** 18;
```

### Method - `unstake(uint256 _safEthAmount)`:
- call `derivatives[i]` 2 time in loop

```solidity
contracts\SafEth\derivatives\SafEth.sol

113:        for (uint256 i = 0; i < derivativeCount; i++) {
114:            // withdraw a percentage of each asset based on the amount of safETH
115:            uint256 derivativeAmount = (derivatives[i].balance() *
116:                _safEthAmount) / safEthTotalSupply;
117:            if (derivativeAmount == 0) continue; // if derivative empty ignore
118:            derivatives[i].withdraw(derivativeAmount);
119:        }
```

### Method - `rebalanceToWeights()`:
- variable `derivativeCount` 2 time in method
- call `derivatives[i]` 3 time in loop
- call `derivatives[i].balance` 2 time in loop
- variable `totalWeight` every iteration
- variable `weights[i]` 2 time in loop

```solidity
contracts\SafEth\derivatives\SafEth.sol

140:        for (uint i = 0; i < derivativeCount; i++) {
147:        for (uint i = 0; i < derivativeCount; i++) {

140:        for (uint i = 0; i < derivativeCount; i++) {
141:            if (derivatives[i].balance() > 0)
142:                derivatives[i].withdraw(derivatives[i].balance());
143:        }

150:                totalWeight;

148:           if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
149:            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
150:                totalWeight;
```
### Method - `rebalanceToWeights()`:
- variable `derivativeCount` 2 time in method
- call `derivatives[i]` 3 time in loop
- call `derivatives[i].balance` 2 time in loop
- variable `totalWeight` every iteration
- variable `weights[i]` 2 time in loop

```solidity
contracts\SafEth\derivatives\SafEth.sol

140:        for (uint i = 0; i < derivativeCount; i++) {
147:        for (uint i = 0; i < derivativeCount; i++) {

140:        for (uint i = 0; i < derivativeCount; i++) {
141:            if (derivatives[i].balance() > 0)
142:                derivatives[i].withdraw(derivatives[i].balance());
143:        }

150:                totalWeight;

148:           if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
149:            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
150:                totalWeight;
```

### Method - `addDerivative(address _contractAddress, uint256 _weight)`:
- variable `derivativeCount` 5 time in method

```solidity
contracts\SafEth\derivatives\SafEth.sol

186:         derivatives[derivativeCount] = IDerivative(_contractAddress);
187:         weights[derivativeCount] = _weight;
188:         derivativeCount++;
189: 
190:         uint256 localTotalWeight = 0;
191:         for (uint256 i = 0; i < derivativeCount; i++)
192:             localTotalWeight += weights[i];
193:         totalWeight = localTotalWeight;
194:         emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
195:     }
```

## Redundant loop in `adjustWeight`, `addDerivative`
In the `adjustWeight` and `addDerivative` function, the loop for calculating the `localTotalWeight` can be optimized by just updating the `totalWeight` based on the change in weight rather than looping through all the derivatives.
```solidity
contracts\SafEth\derivatives\SafEth.sol

169:        weights[_derivativeIndex] = _weight;
170:        uint256 localTotalWeight = 0;
171:        for (uint256 i = 0; i < derivativeCount; i++)
172:            localTotalWeight += weights[i];
173:        totalWeight = localTotalWeight;
174:        emit WeightChange(_derivativeIndex, _weight);

190:        uint256 localTotalWeight = 0;
191:        for (uint256 i = 0; i < derivativeCount; i++)
192:            localTotalWeight += weights[i];
193:        totalWeight = localTotalWeight;
```
Example solution:
```solidity
// adjustWeight
        totalWeight = totalWeight - weights[_derivativeIndex] + _weight;
        weights[_derivativeIndex] = _weight;
        emit WeightChange(_derivativeIndex, _weight);

// addDerivate
        totalWeight += _weight;
```

## Redundant iterations when `ethAmountToRebalance` equal ZERO
When calling the `rebalanceToWeights()` method, if `ethAmountToRebalance` is equal to ZERO, going through the loop is redundant as every iteration in the loop will be skipped
```solidity
contracts\SafEth\derivatives\SafEth.sol

145:         uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
146: 
147:         for (uint i = 0; i < derivativeCount; i++) {
148:             if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
149:             uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
150:                 totalWeight;
151:             // Price will change due to slippage
152:             derivatives[i].deposit{value: ethAmount}();
153:         }
```

It's recommended to move the condition `ethAmountToRebalance == 0` outside the loop, for example:
```solidity
        uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
        if (ethAmountToRebalance != 0){
            for (uint i = 0; i < derivativeCount; i++) {
                if (weights[i] == 0) continue;
                uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                    totalWeight;
                // Price will change due to slippage
                derivatives[i].deposit{value: ethAmount}();
            }
        }
```


## Calculation of static values
Dynamic calculation of values ​​with a known result should be avoided for better readability (reduction count of magic numbers) and lower gas prices

### Instances:
```solidity
contracts\SafEth\derivatives\Reth.sol

44:         maxSlippage = (1 * 10 ** 16); // 1%

171:        uint rethPerEth = (10 ** 36) / poolPrice();

173:        uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
174:                 ((10 ** 18 - maxSlippage))) / 10 ** 18);

214:        RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
215:         else return (poolPrice() * 10 ** 18) / (10 ** 18);

241:         return (sqrtPriceX96 * (uint(sqrtPriceX96)) * (1e18)) >> (96 * 2);
```
```solidity
contracts\SafEth\derivatives\SfrxEth.sol

38:        maxSlippage = (1 * 10 ** 16); // 1%

74:        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
75:            (10 ** 18 - maxSlippage)) / 10 ** 18;

112:        uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
113:            10 ** 18
114:        );
115:        return ((10 ** 18 * frxAmount) /
116:            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
```
```solidity
contracts\SafEth\derivatives\WstEth.sol

35:        maxSlippage = (1 * 10 ** 16); // 1%

60:        uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;

87:        return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
```
```solidity
contracts\SafEth\SafEth.sol

54:        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
55:        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum

72:            underlyingValue +=
73:                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74:                    derivatives[i].balance()) /
75:                10 ** 18;

79:        if (totalSupply == 0)
80:            preDepositPrice = 10 ** 18; // initializes with a price of 1
81:        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;


92:            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
93:                depositAmount
94:            ) * depositAmount) / 10 ** 18;

98:        uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```

### example solution

* `10 ** 18` -> `1 ether`;
* `1 * 10 ** 16` -> `1e16` | `0.01 ether` | move to constants `ONE_PERCENTAGE`;
* `5 * 10 ** 17` -> `0.5 ether`
* `200 * 10 ** 18` -> `200 ether`
* `96 * 2` -> `192`

## Use a more recent version of solidity

In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

In 0.8.18 Added new optimization

In 0.8.19 better finding and organizing duplicate assembly code


## Save gas with the use of the import statement
With the import statement, it saves gas to specifically import only the parts of the contracts, not the complete ones.
Example:
```solidity
import {contract1 , contract2} from "filename.sol";
```
*Instances*:
```solidity
contracts\SafEth\derivatives\WstEth.sol

5: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

```


## Using a positive conditional flow to save a NOT opcode
```solidity 
contracts\SafEth\derivatives\Reth.sol
170:         if (!poolCanDeposit(msg.value)) 
```