### Low Risk Issues List

| Number | Issues Details                     | Instances |
| :----: | :--------------------------------- | :-------: |
| [L-01] | Unspecific compiler version pragma |     4     |

Total: 1 issue

#### [L-01] Unspecific compiler version pragma

The compiler version specified for a contract should be locked to the version it has been tested the most with. Locking the pragma helps ensure that contracts do not accidentally get deployed using, for example, the latest compiler which may have higher risks of undiscovered bugs. Contracts may also be deployed by others and the pragma indicates the compiler version intended by the original authors. [locking-pragmas/](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

For example:

```solidity
// bad
pragma solidity ^0.8.13;

// good
pragma solidity 0.8.13;
```

1. [SafEth.sol Line 2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2)
2. [Reth.sol Line 2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2)
3. [SfrxEth.sol Line 2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L2)
4. [WstEth.sol Line 2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L2)

### Non-Critical Issues List

| Number | Issues Details | Instances |
| :-: | :-- | :-: |
| [N-01] | Readability and maintainability | 1 |
| [N-02] | Function grouping and ordering | 3 |
| [N-03] | Maximum Line Length | 2 |
| [N-04] | Constants should be defined rather than using magic numbers | 1 |
| [N-05] | Control structure style | 1 |

Total: 5 issues

#### [N-01] Readability and maintainability

Some code could be made easier to read and maintain by changing the syntax.

```solidity
File: contracts/SafEth/SafEth.sol

// before
54:    minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
55:    maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum

// after
54:    minAmount = 1 ether / 2; // initializing with .5 ETH as minimum
55:    maxAmount = 200 ether;   // initializing with 200 ETH as maximum
```

#### [N-02] Functions should be grouped according to their visibility and ordered per the Solidity Style Guide

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. See [Order of Functions](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions).

1. In [Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol):
   1. `receive()` should come right after the constructor.
   2. The external functions `setMaxSlippage()`, `withdraw()`, `deposit()` should come right after `initialize()`.
   3. The public function `ethPerDerivative()`, `balance()` should come right after the last external function.
2. In [SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol):
   1. `receive()` should come right after the constructor.
   2. The external functions `setMaxSlippage()`, `withdraw()`, `deposit()` should come right after `initialize()`.
3. In [WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol):
   1. `receive()` should come right after the constructor.
   2. The external functions `setMaxSlippage()`, `withdraw()`, `deposit()` should come right after `initialize()`.

#### [N-03] The maximum suggested line length is 120 characters per the Solidity Style Guide

Several lines are longer than this, making the code unnecessarily difficult to read and maintain. See [Maximum Line Length](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length).

The following lines exceed 120 characters.

1. [SafEth.sol Line 160](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L160)
2. [Reth.sol Line 142](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L142)

#### [N-04] Constants should be defined rather than using magic numbers

1. [Reth.sol Line 180](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L180) Magic number: `500`

   ```solidity
   177:  uint256 amountSwapped = swapExactInputSingleHop(
   178:      W_ETH_ADDRESS,
   179:      rethAddress(),
   180:      500,
   181:      msg.value,
   182:      minOut
   183:  );
   ```

#### [N-05] Control structure style

Per the [Solidity Style Guide](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures), "For control structures whose body contains a single statement, omitting the braces is ok if the statement is contained on a single line." There is some code here that does not follow this guide. It would be best to follow the guide to make the code more readable and maintainable.

1. [SafEth.sol Line 71](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71) Since the body of this `for` loop spans multiple lines, curly braces should be added to clearly delineate the start and finish of the code executed in the loop.
   ```solidity
    71:  for (uint i = 0; i < derivativeCount; i++)
    72:      underlyingValue +=
    73:            (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
    74:                derivatives[i].balance()) /
    75:            10 ** 18;
   ```
