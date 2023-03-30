## Pack global storage variables of SafETH

The current storage takes 5 slots:
```solidity
bool public pauseStaking;
bool public pauseUnstaking;
uint256 public derivativeCount;
uint256 public totalWeight;
uint256 public minAmount;
uint256 public maxAmount;
```

This is excessive, the types can be much smaller:
- `derivativeCount` can be `uint16` there won't be more than 65 thousand derivatives
- `totalWeight` can be `uint32`, the weights scale of 4 billion should be accurate enough
- `minAmount` and `maxAmount` can be `uint96`, the total supply of ETH in the ecosystem is a number that can fit in 87 bits, using 96 is more than safe.

The optimized storage fits exactly in 256 bits and takes only 1 slot:
```solidity
bool public pauseStaking;
bool public pauseUnstaking;
uint16 public derivativeCount;
uint32 public totalWeight;
uint96 public minAmount;
uint96 public maxAmount;
```
If it's necessary, the storage can be reduced by 1 more byte if `pause` variables are merged into a bitmap.

## Pack per-derivative storage variables of SafETH

Each derivative takes at least 3 slots:
```solidity
mapping(uint256 => IDerivative) public derivatives; // derivatives in the system
mapping(uint256 => uint256) public weights; // weights for each derivative
```
The third slot is the `maxSlippage` variable stored by each derivative.

- `weights` can be reduced to `uint32` as mentioned in the global storage packing finding
- `maxSlippage` can be reduced to `uint64`, because it only deals with numbers on a scale from `0` (no slippage allowed) to `10^18` (any slippage allowed), which needs just 60 bits. Going further this value doesn't need to be stored in each derivative separately, it can be passed in `deposit` and `withdraw` functions. It's more flexible this way, keeps the configuration in one place, simplifies the derivatives implementation and makes events cleaner, currently derivatives don't emit anything even though their internal state is changed.

The optimized per-derivative storage fits exactly in 256 bits and takes 1 slot:
```solidity
struct DerivativeStorage {
    IDerivative derivative;
    uint32 weight;
    uint64 maxSlippage;
}
mapping(uint256 => DerivativeStorage) public derivatives;
```

## Cache the current balance in `stake`

In the current implementation of `stake` in https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L73 `derivatives[i].balance()` is called twice in a row, but its result could be stored in a local variable and reused.

```solidity
uint256 balance = derivatives[i].balance();
underlyingValue += (derivatives[i].ethPerDerivative(balance) * balance) / 10 ** 18;
```