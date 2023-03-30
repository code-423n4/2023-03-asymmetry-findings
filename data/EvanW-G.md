[G-01] Merge the two for-loop as one in stake()

In SafEth.sol, the stake() method is running the same for-loop twice which is unecessary. Do all the things in 1 loop could save 132 gas.

```solidity
contracts/SafEth/SafEth.sol
71: for (uint i = 0; i < derivativeCount; i++)
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                    derivatives[i].balance()) /
                10 ** 18;

84: for (uint i = 0; i < derivativeCount; i++) {
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
```

Recommandation:

```solidity
for (uint i = 0; i < derivativeCount; i++) {
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                    derivatives[i].balance()) /
                10 ** 18;
            uint256 weight = weights[i];
            IDerivative derivative = derivatives[i];
            if (weight != 0) {
	            uint256 ethAmount = (msg.value * weight) / totalWeight;
	
	            // This is slightly less than ethAmount because slippage
	            uint256 depositAmount = derivative.deposit{value: ethAmount}();
	            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
	                depositAmount
	            ) * depositAmount) / 10 ** 18;
	            totalStakeValueEth += derivativeReceivedEthValue;
						};
        }
```

[G-02] zero value checking of “_safEthAmount”

Add a require statement for checking the _safEthAmount value could prevent users wasting gas because of incorrect input

```solidity
contracts/SafEth/SafEth.sol
108: function unstake(uint256 _safEthAmount) external {
```

[G-03] ****Use nested if and, avoid multiple check combinations****

```solidity
contracts/SafEth/SafEth.sol
147: for (uint i = 0; i < derivativeCount; i++) {
            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                totalWeight;
            // Price will change due to slippage
            derivatives[i].deposit{value: ethAmount}();
        }
```

Recommand:

```solidity
     for (uint i = 0; i < derivativeCount; i++) {
            // if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
            if (weights[i] != 0 ) {
                if (ethAmountToRebalance != 0) {
                    uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                        totalWeight;
                    // Price will change due to slippage
                    derivatives[i].deposit{value: ethAmount}();
                }
            }
            continue;
        }
```

[G-06] Avoid always reading ****`state variable`.**

In function “addDerivative”, it’s better to create and memory variable for derivativeCount rather than repeatedly reading state variable

```solidity
contracts/SafEth/SafEth.sol
191: derivatives[derivativeCount] = IDerivative(_contractAddress);
192: weights[derivativeCount] = _weight;
196: for (uint256 i = 0; i < derivativeCount; i++)
199: emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
```

[G-05] ****Avoid using `state variable` in emit (130 gas)**

Emit the memory variable or calldata variable instead.

```solidity
contracts/SafEth/SafEth.sol

199: emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
221: emit ChangeMinAmount(minAmount);
230: emit ChangeMaxAmount(maxAmount);
239: emit StakingPaused(pauseStaking);
248: emit UnstakingPaused(pauseUnstaking);
```

[G-06] Some imports are not used.

```solidity
contracts/SafEth/SafEth.sol
4: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
5: import "../interfaces/IWETH.sol";
6: import "../interfaces/uniswap/ISwapRouter.sol";
7: import "../interfaces/lido/IWStETH.sol";
8: import "../interfaces/lido/IstETH.sol";

contracts/SafEth/derivativrs/Reth.sol
5: import "../../interfaces/frax/IsFrxEth.sol";

```

[G-07] No need to create memory variable if it is being used once only

```solidity
contracts/SafEth/derivativrs/WstEth.sol
79: uint256 wstEthAmount = wstEthBalancePost - wstEthBalancePre;
```