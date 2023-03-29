[N-1] Constants in comparisons should appear on the left side
===

[It's a mechanism to avoid mistakes like this](https://stackoverflow.com/a/370373)

```solidity
File: contracts/SafEth/SafEth.sol
79:         if (totalSupply == 0)
```
```solidity
File: contracts/SafEth/SafEth.sol
87:             if (weight == 0) continue;
```
```solidity
File: contracts/SafEth/SafEth.sol
117:             if (derivativeAmount == 0) continue; // if derivative empty ignore
```
```solidity
File: contracts/SafEth/SafEth.sol
148:             if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
```
```solidity
File: contracts/SafEth/SafEth.sol
64:         require(pauseStaking == false, "staking is paused");
```
```solidity
File: contracts/SafEth/SafEth.sol
109:         require(pauseUnstaking == false, "unstaking is paused");
```
[N-2] Events that mark critical parameter changes should contain both the old and the new value
===
Example
---
before:
```solidity
File: contracts/SafEth/SafEth.sol
174:         emit WeightChange(_derivativeIndex, _weight);
```
after:
```solidity
File: contracts/SafEth/SafEth.sol
174:         emit WeightChange(_derivativeIndex, oldWeight, newWeight);
```
other instances:
----
```solidity
File: contracts/SafEth/SafEth.sol
216:         emit ChangeMinAmount(minAmount);
```
```solidity
File: contracts/SafEth/SafEth.sol
225:         emit ChangeMaxAmount(maxAmount);
```
[N-3] Missing event for critical parameter change
===
```solidity
File: contracts/SafEth/derivatives/Reth.sol
58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {
59:         maxSlippage = _slippage;
60:     }
```
Similarly in `SfrxEth.sol` and `WstEth.sol`
This method can be called directly on a derivative contract or via a `SafETH.setMaxSlippage(uint, uint)` contract (where the event is emitted). 
If called directly, there will be no event. 
It's better to move the event to the derivative implementation, where we also have access to the changed value. 
The event could look like this:
```solidity
emit SetMaxSlippage(oldMaxSlippage, newMaxSlippage);
```
[N-4] Common code should be refactored
===
Each derivative implementation will include the identical `setMaxSlippage(uint256 slippage)` function. This can be extracted to the base contract and overridden in a specific implementation if needed. This will definitely affect the simplicity of the code and the maintenance of the project with the increasing number of derivatives.
```solidity
File: contracts/SafEth/derivatives/Reth.sol
58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {
59:         maxSlippage = _slippage;
60:     }
```
Example implementation:
---
```solidity
abstract contract BaseDerivative is IDerivative{

    uint256 public maxSlippage;
    
    function setMaxSlippage(uint256 _slippage) external virtual {
        maxSlippage = _slippage;
    }
}
```
[N-5] Function ordering does not follow the Solidity style guide
===
According to the [Solidity style guide](https://docs.soliditylang.org/en/v0.8.13/style-guide.html#order-of-functions), functions should be laid out in the following order:

* constructor
* receive function (if exists)
* fallback function (if exists)
* external
* public
* internal
* private

None of the contracts meet the guidelines.