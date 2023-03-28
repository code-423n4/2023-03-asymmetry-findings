# Low Issues

|  |	Issues Details |
|---|---|
| [Low-1] | Loss of Precision due to rounding  |
| [Low-2] | Function Parameters Without Bounds |
| [Low-3] | Owner can renounce Ownership |
| [Low-4] | Lack of zero address checks |

## [Low-1] Loss of Precision due to rounding
In the `stake` function, the ethAmount is calculated by `(msg.value * weight) / totalWeight` at line 88. Due to a potential loss of precision when rounding, there is a possibility that some wei may remain in the `SafEth` contract.

For instance, consider the following initial state:
`msg.value` = 200;
`weights[0]` = 10;
`weights[1]` = 20;
`weights[2]` = 30;
`totalWeight` = 60;

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84-L88
```solidity=84
    for (uint i = 0; i < derivativeCount; i++) {
        uint256 weight = weights[i];
        IDerivative derivative = derivatives[i];
        if (weight == 0) continue;
        uint256 ethAmount = (msg.value * weight) / totalWeight;
```

In this scenario, the `ethAmount` values would be `33`, `66`, and `99` for the three iterations of the loop, respectively. Therefore, there would be small wei left in the `SafETH` contract since `200 - (33 + 66 + 99) = 2` (without slippage).

### Recommendation

Use a different mechanism to allocate funds, instead of calculating ethAmount based on weights, the contract could use a fixed price for each derivative or allow users to specify the exact amount of ETH they want to allocate to each derivative.

## [Low-2] Function Parameters Without Bounds
The function can be called with any input value for those parameters, regardless of whether they are valid or reasonable. This can lead to unexpected behavior or even security vulnerabilities if the function does not properly validate or sanitize its input parameters.
For example:

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202-L208
```solidity=202
    function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
    ) external onlyOwner {
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
    }
```

### Recommendation
Define limits or constraints on function parameters to prevent unintended consequences.

## [Low-3] Owner can renounce Ownership
Renouncing ownership results in the contract being left without an owner, removing any functionality that is only available to the owner.

### Recommendation
Reimplementing the function to disable it or clearly stating whether it is part of the contract design.

## [Low-4] Lack of zero address checks
If the variable get configured with address zero, failure to immediately reset the value can result in unexpected behavior for the project.

### Reccommendation
Add zero address checks.