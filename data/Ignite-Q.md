# Low Issues

|  |	Issues Details | Instances |
|---|---|---|
| [Low-1] | Loss of Precision due to rounding  | 1 |
| [Low-2] | Function Parameters Without Bounds | 1 |
| [Low-3] | Missing deadline checks in `swapExactInputSingleHop()` function | 1 |
| [Low-4] | Lack of zero address checks | 4 | 
| [Low-5] | Pragma Float | All Contracts |

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
Transfer the remaining wei back to the sender.

Otherwise, use a different mechanism to allocate funds, instead of calculating `ethAmount` based on weights, the contract could use a fixed price for each derivative or allow users to specify the exact amount of ETH they want to allocate to each derivative.

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

## [Low-3] Missing deadline checks in `swapExactInputSingleHop()` function

The `swapExactInputSingleHop()` function lacks a deadline for its actions that execute swaps on the `deposit()` function. This missing feature can result in pending transactions being executed later, potentially leading to outdated slippage.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L83-L89
```solidity
    function swapExactInputSingleHop(
        address _tokenIn,
        address _tokenOut,
        uint24 _poolFee,
        uint256 _amountIn,
        uint256 _minOut
    ) private returns (uint256 amountOut) {
```

### Recommendation
Adding a `deadline` parameter to function which perform a swap.

## [Low-4] Lack of zero address checks
If the variable get configured with address zero, failure to immediately reset the value can result in unexpected behavior for the project.

The following table contains all instances where variables are assigned an address:

| Contract | Line | Link to Code |
| -------- | -------- | ---- |
| SafEth   | 186 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L186 |
| Reth     | 43 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L43 |
| SfrxEth     | 37 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L37 |
| WstEth     | 34 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L34 |

### Recommendation
Add zero address checks.

## [Low-5] Pragma Float
All the contracts being considered do not specify a specific pragma version. However, the hardhat config file uses solidity version 0.8.13 for deployment, which is known to have a well-known bug (https://docs.soliditylang.org/en/v0.8.19/bugs.html).

### Recommendation
Setting a specific pragma version is an effective way to prevent contracts from being deployed with an outdated compiler version. I recommend using the latest stable version of Solidity, which is 0.8.19, instead of relying on an older version like 0.8.13 that is known to have bugs.

# Non-Critical Issues

| | Issues | Instances |
| -------- | -------- | -------- |
| NC-1     | Using `uint256` rather than `uint`     | 14 |

## [NC-1] Using `uint256` rather than `uint`

Using `uint256` instead of `uint` is for clarity and consistency. Using `uint256` indicates that you are specifically declaring an unsigned integer of 256 bits, while using `uint` may be less clear and may lead to confusion with other integer types that have a different number of bits.

The following table contains all instances where `uint` is used:

| Contract | Line | Link to Code |
| -------- | -------- | ---- |
| SafEth   | 26 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L26 |
| SafEth   | 27 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L27 |
| SafEth   | 28 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L28 |
| SafEth   | 31 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L31 |
| SafEth   | 32 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L32 |
| SafEth   | 71 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71 |
| SafEth   | 84 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84 |
| SafEth   | 92 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L92 |
| SafEth   | 140 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140 |
| SafEth   | 147 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147 |
| SafEth   | 203 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L203 |
| SafEth   | 204 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L204 |
| Reth     | 171 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L171 |
| Reth     | 241 | https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L241 |

### Recommendation
Using `uint256` rather than `uint`.