## Reorder `SafEthStorage` variables for lesser storage accesses

Considering that `stake()` is reasonably expected to be the most-called function, it currently accesses 5 storage slots while reading the following variables:

* `pauseStaking`
* `derivativeCount`
* `totalWeight`
* `minAmount`
* `maxAmount`

However, both `derivativeCount` and `totalWeight` are not expected to grow much, so they can be packed along with `pauseStaking`, and for the case of `minAmount` and `maxAmount` both current values hardcoded in the contract can fit in `uin128` to be packed together.

Concretely, the arrangement proposed is the following:

```solidity
contract SafEthStorage {
    bool public pauseStaking; // true if staking is paused
    bool public pauseUnstaking; // true if unstaking is pause
    uint120 public derivativeCount; // amount of derivatives added to contract
    uint120 public totalWeight; // total weight of all derivatives (used to calculate percentage of derivative)
    uint128 public minAmount; // minimum amount to stake
    uint128 public maxAmount; // maximum amount to stake
    ...
}
```