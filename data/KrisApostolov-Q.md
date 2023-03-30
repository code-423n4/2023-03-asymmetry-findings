# QA report

## Disclaimer

Some of the examples may contain truncated versions of the original code. There may also be `@audit` tags, which denote the actual place affected

# Table of contents

### Low severity findings

|  | Issue | Instances |
| --- | --- | --- |
| [L-1] | Lack of balance check allows the unstake function to be called without a corresponding balance or amount | 1 |
| [L-02] | Not following the CEI pattern can lead to a reentrancy | 1 |
| [L-03] | Slight precision loss occurring in the calculations | 2 |

### [L-01] - Lack of balance check allows the `unstake` function to be called without a corresponding balance or amount

The **unstake** function in the **[SafEth](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)** contract does not implement any balance checks whatsoever, which enables anyone to call the function with a **`_safEthAmount` = 0** without any reverts. The function will execute everything like normal thus wasting gas and opening the contract to other potential vulnerabilities.

### Instances

**File** **[./contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol):**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108-L129) - `unstake()`**

### Recommendations

Consider adding 2 types of checks in the **unstake** function:

1. **A** **`require(_safEthAmount ≠ 0, “error message”);` statement or similar.**
2. **A `require(balanceOf(msg.sender) ≥ _safEthAmount, “error message”)`** **statement**  **or similar.**

### [L-02] - Not following the CEI pattern can lead to a reentrancy

The **unstake** function in the **[SafEth](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)** contract does not follow the checks-effects-interactions pattern and goes straight to calling an external contract (a derivative) to withdraw amount the user wants before burning the user’s receipt tokens. This creates an opportunity in which someone can find a way to re-enter into the function before their tokens get burned, which could lead to fund theft.

### Instances

**File** [**./contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol):**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108-L129) - `unstake()`**

### Recommendations

Consider burning the receipt tokens of the user before actually withdrawing any eth from the derivatives, thus minimizing the risk of reentrancy.

### [L-03] - Slight precision loss occurring in the calculations

### Instances

**File [./contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol) 1 instance:**

[**[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173-L174) -** **`((((rethPerEth * msg.value) / 10 ** 18) * ((10 ** 18 - maxSlippage))) / 10 ** 18)`**

The **Uniswap [minimum amount calculation](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173-L174)** in the **`deposit()`** of the **Rocket Eth** derivative produces a 1-2 WEI precision loss in half of the times. It happens because division is done before multiplication, which causes loss in precision. This effectively locks that small amount of eth each time a deposit in the derivative has been made.

### Recommendations

Consider re-factoring the equation to the following: **`(((rethPerEth * msg.value * (1e18 - maxSlippage)) / (1e36)))`.**

**File [./contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol) 1 instance:**

[**[1]**](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75) **-**

The **Curve pool [minimum amount calculation](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75)** in the **`withdraw()`** of the **Sfrx Eth** derivative produces a 1-2 WEI precision loss in half of the times. It happens because division is done before multiplication, which causes loss in precision. This effectively locks that small amount of eth each time a deposit in the derivative has been made.

### Recommendations

Consider re-factoring the equation to the following: `**(_amount * ethPerDerivative(_amount) * (1e18 - maxSlippage)) / (1e36)**`

### Non-Critical findings

|  | Issue | Instances |
| --- | --- | --- |
| [N-01] | Insufficient coverage | The protocol derivatives |
| [N-02] | Use scientific notation instead of 10 ** x | The whole protocol |
| [N-03] | Use specific imports instead of whole file imports for cleanliness | The whole protocol |
| [N-04] | Omit the parameter name of an unused param in the specific scenario | 2 |
| [N-05] | Floating pragma | The whole protocol |
| [N-06] | Create getter functions for rocketDepositPool and rocketDAOProtocolSettingsDeposit | 2 |
| [N-07] | Use the rethAddress() function instead of  re-implementing the same code in another function | 1 |
| [N-08] | Unnecessary else block | 1 |
| [N-09] | Remove unnecessary calculation | 1 |

### [N-01] - **Insufficient coverage**

The derivative contracts that have been implemented do not have 100% code coverage.

Having 100% code coverage is a best practice in terms of security.

### Recommendations

Consider testing all of the functions of the derivative contracts until 100%.

### [N-02] - Use scientific notation instead of `x * 10 ** y`

### Instances

 

Add all of the expressions where **`**`** is used.

### Recommendations

Instead of using the `x * **10 ** y**` syntax use the scientific notation syntax: `**xey**`

**Example:**

`**x = 1**`, **`y = 18`.**

**Before:** **`1 * 10 ** 18` After:** **`1e18`.**

### [N-03] - Use specific imports instead of whole file imports for cleanliness

In order to keep the codebase cleaner and not know where each thing comes from all imports should be explicit.

### Instances

The whole protocol.

### Recommendations

Consider using the following syntax for imports:

```solidity
import {Initializable} from "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
```

### [N-04] - Omit the parameter name of an unused param in the specific scenario

The **`ethPerDerivative()`** functions in the **`WstEth`** and **`SfrxEth`** contracts contain an unused parameter that is there because of the way the function of each derivative gets called in the stake function of the **`SafEth`** contract.

### Instances

**File [./contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol) 1 instance:**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L111-L117)**

**File [./contracts/SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol) 1 instance:**

**[[2]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86-L88)**

### Recommendations

Consider removing the **`_amount`** name of the parameter and leaving out only the **`uint256`** type so the compiler does not revert when the function gets called with a parameter.

### [N-05] - Floating pragma

When intended for deployment a contract should always use a locked solidity version in order to make sure that the code is optimized and tested for its solidity version.
Check **[this](https://swcregistry.io/docs/SWC-103)** for more details.

### Instances

**File** **[./contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)**

**File [./contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol)**

**File [./contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol)**

**File [./contracts/SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol)**

### Recommendations

Consider changing the compiler version of all of the files from `pragma solidity ^0.8.13` to `pragma solidity 0.8.13`.

### [N-06] - Create getter functions for **`rocketDepositPool`** and **`rocketDAOProtocolSettingsDeposit`**

The following blocks just pollute the business logic of the Rocket Eth derivative and make it hard to understand what is happening in the [**poolCanDeposit()**](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L120-L150) function.

### Instances

**File [./contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol):**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L121-L127) -** **`rocketDepositPoolAddress`**

[**[2]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L132-L141) -** **`rocketProtocolSettingsAddress`**

### Recommendations

Consider isolating them in the same manner as the **[rethAddress()](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L66-L73)** function.

### [N-07] - Use the **`rethAddress()`** function instead of  re-implementing the same code in another function

The code block referenced below contains exactly the same code as the **`rethAddress()`** function. There is no point it doing so instead of calling the function directly below.

### Instance

**File [./contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol):**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L187-L193)**

### Recommended

Consider replacing the block with the invocation of the **`rethAddress()`** function.

### [N-08] - Unnecessary else block

Unnecessary else block that does no function

### Instance

**File [./contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol):**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L186-L203)**

### Recommendations

Consider removing the else block and unnesting the code

### [N-09] - Remove unnecessary calculation

The following calculation in the Rocket Eth derivative contract is excessive: `**(poolPrice() * 10 ** 18) / (10 ** 18)**`

### Instance

**File [./contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol):**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215) - `(poolPrice() * 10 ** 18) / (10 ** 18)`**

### Recommendation

Simplify the equation to only **`poolPrice()`**