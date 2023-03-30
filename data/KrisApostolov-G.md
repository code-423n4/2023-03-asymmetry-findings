# Gas Report

## Disclaimer

Some of the examples may contain truncated versions of the original code. There may also be `@audit` tags, which denote the actual place affected

## Table of contents

|  | Issue | Instances | Estimated savings |
| --- | --- | --- | --- |
| [G-01] | Functions guaranteed to revert, when called by normal users should be marked private to save on gas | 21 | 441 |
| [G-02] | Use unchecked blocks on variables that are guaranteed to not overflow or underflow | 1 | 126 |
| [G-03] | Unnecessary external call(s) | 2 | ^3000 |
| [G-04] | Unnecessary variable allocation | 3 | 39 |
| [G-05] | Hashes can be cached into into immutable variables | 6 | 2772 |
| [G-06] | Refactor the weight calculation | 1 | ^327 |
| [G-07] | Saving storage variable into memory saves gas when variable is called more than once | 11 | 1067 |
| [G-08] | Saving the output of a function when possible to save on gas | 2 | ^5000 |
| [G-09] | Optimize names to save gas | The whole protocol | ^500 |
| [G-10] | Emitting an event with a storage variable wastes gas | 5 | 650 |
| [G-11] | address.call can be optimized with assembly | 5 | - |

### **Estimated savings: ^14500 gas**

## [G-01] - Functions guaranteed to revert, when called by normal users should be marked private to save on gas

In the event that a function modifier or a "require" statement like `onlyOwner` is utilized, an attempt by a regular user to make a call to the function will result in the function being reverted. If the function is marked as "payable," the cost of gas for legitimate callers will decrease, as the compiler will no longer need to include checks to determine if payment was provided. This means that the additional opcodes that are typically avoided, including `CALLVALUE(2)`, `DUP1(3)`, `ISZERO(3)`, `PUSH2(3)`, `JUMPI(10)`, `PUSH1(3)`, `DUP1(3)`, `REVERT(0)`, `JUMPDEST(1)`, and `POP(2)`, which typically cost an average of around **21 gas** per call to the function, will not be included. The following instances even include the proxy initializers as they are not an exception to the gas saving.

### **Instances:**

**File** **[./contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol) - 9 instances:**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L48-L51) - `initialize()`**

**[[2]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138) - `rebalanceToWeights()`**

[**[3]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L168) - `adjustWeight()`**

**[[4]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-L185) - `addDerivative()`**

**[[5]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202-L205) - `setMaxSlippage()`**

**[[6]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214) - `setMinAmount()`**

**[[7]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223) - `setMaxAmount()`**

**[[8]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232) - `setPauseStaking()`**

**[[9]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241) - `setPauseUnstaking()`**

**File [./contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol) - 4 instances:**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L42) - `initialize()`**

**[[2]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58) - `setMaxSlippage()`**

**[[3]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107) - `withdraw()`**

**[[4]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L156) - `deposit()`**

**File [./contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol) - 4 instances:**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L36) - `initialize()`**

**[[2]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51) - `setMaxSlippage()`**

**[[3]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60) - `withdraw()`**

**[[4]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L94) - `deposit()`**

**File [./contracts/SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol) - 4 instances:**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L33) - `initialize()`**

**[[2]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48) - `setMaxSlippage()`**

**[[3]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56) - `withdraw()`**

**[[4]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L73) - `deposit()`**

### Recommendations

Consider adding the payable modifier to the mentioned functions.

**This will decrease the gas usage of each function by 21.**

## [G-02] - Use unchecked blocks on variables that are guaranteed to not overflow or underflow

In the current situation, the projected amount of derivatives that the protocol will support is around 5 derivatives as per Toshi. This means that it is highly unlikely for the **`derivativeCount`** variable to overflow.

### Instances:

**File [./contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol) - 1 instance:**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L188) - `derivativeCount++;`**

### Recommendations

Consider doing the following:

**`derivativeCount++;` to `unchecked { ++derivativeCount; }`**

**Decreases gas usage by ~126.**

## [G-03] - Unnecessary external call(s)

Certain functions call external contracts without it being needed.

### Instances:

**File [./contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol) - 2 instances:**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60)** - 

```solidity
SfrxEth.sol
function withdraw(uint256 _amount) external onlyOwner {
// The redeem function returns the withdrawn amount, which is equal to the frxEthBalance variable
        IsFrxEth(SFRX_ETH_ADDRESS).redeem(
            _amount,
            address(this),
            address(this)
        );
        uint256 frxEthBalance = IERC20(FRX_ETH_ADDRESS).balanceOf(
            address(this)
        );
        IsFrxEth(FRX_ETH_ADDRESS).approve(
            FRX_ETH_CRV_POOL_ADDRESS,
            frxEthBalance
        );

        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
            (10 ** 18 - maxSlippage)) / 10 ** 18;

        IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).exchange(
            1,
            0,
            frxEthBalance,
            minOut
        );
        // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
        require(sent, "Failed to send Ether");
    }
```

The `redeem` function returns the withdrawn amount.

### Recommendations

Consider removing lines [66-68](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L66-L68) altogether and assigning the `uint256 frxEthBalance` variable to the redeem function.

**This can save upwards of 1000 gas.**

**[[2]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L94) -** 

```solidity
Sfrx.sol
function deposit() external payable onlyOwner returns (uint256) {
        IFrxETHMinter frxETHMinterContract = IFrxETHMinter(
            FRX_ETH_MINTER_ADDRESS
        );
        uint256 sfrxBalancePre = IERC20(SFRX_ETH_ADDRESS).balanceOf(
            address(this)
        );
        frxETHMinterContract.submitAndDeposit{value: msg.value}(address(this));
        uint256 sfrxBalancePost = IERC20(SFRX_ETH_ADDRESS).balanceOf(
            address(this)
        );
        return sfrxBalancePost - sfrxBalancePre;
    }
```

Line [101](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L101) returns `sfrxBalancePost - sfrxBalancePre`.

### Recommendations

Consider removing the lines [98-100](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L98-L100) and [102-104](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L102-L104) entirely and directly return the result of line  [101](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L101).

**This can save upwards of 2000 gas and can mitigate the risks that external calls bring.**

## [G-04] - Unnecessary variable allocation

### Instances:

**File** **[./contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol) - 2 instances:**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L177) - `deposit()`**

### Recommendations

 ****Instead of assigning the **`uint256 amountSwapped`** variable return the **`swapExactInputSingleHop(W_ETH_ADDRESS, rethAddress(), 500, msg.value, minOut);`** expression directly.

**[[2]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L201-L202) -** **`deposit()`**

### Recommendations

Instead of assigning `rethMinted` variable simply return **`rethBalance2 - rethBalance1`.**

**File [./contracts/SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol) - 1 instance:**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L79-L80) - `deposit()`**

### Recommendations

Instead of declaring both the **`wstEthAmount`** and **`stEthBalancePost`** variables ****directly return the following `return IWStETH(WST_ETH).balanceOf(address(this)) - wstEthBalancePre;` .

**All occurrences save 13 gas per variable.**

## [G-05] - Hashes can be cached into into immutable variables

### Instances:

**File** **[./contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol) - 6 instances:**

[**[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L69-L71)   [[2]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L190-L192)   [[3]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L232-L234) -** `keccak256(abi.encodePacked("contract.address", "rocketTokenRETH"))` **hash.**

[**[4]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L124-L126)  [[5]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L161-L163) -** `abi.encodePacked("contract.address", "rocketDepositPool")` **hash.**

**[[6]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L135-L140) -  `keccak256(abi.encodePacked("contract.address", "rocketDAOProtocolSettingsDeposit"))` hash.**

### Recommendations

Isolate the following instances into immutable storage variables. Assign the hashes’ values to the variables in the `initialize` function. This way it caches the output of the operation and only gas for calling a storage variable is added.

**Saves 462 gas per instance.**

## [G-06] - Refactor the weight calculation

The following expression:

```solidity
for (uint256 i = 0; i < derivativeCount; i++) localTotalWeight += weights[i];
```

is reading `i` times from storage. Which can get pretty expensive over time if multiple derivatives get added and removed. 

### Instances:

**File [./contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol) - 1 instance:**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175) -** `adjustWeight()`

### Recommendations

Consider re-writing the function in the following way:

```solidity
SafEth.sol
function adjustWeight(uint256 _derivativeIndex, uint256 _weight) external onlyOwner {
    uint256 oldWeight = weights[_derivativeIndex]
		weights[_derivativeIndex] = _weight;
    totalWeight = totalWeight + _weight - oldWeight;
    emit WeightChange(_derivativeIndex, _weight);
  }
```

This way the storage reads get minimized to 2 and the gas cost stays constant.

**This saves `i * (100 + 3) - 2 * 100` gas per function call.**

In a case where there are 5 derivatives this can save up to 327 gas

## [G-07] - Saving storage variable into memory saves gas when variable is called more than once

When referencing storage variables more than once it is best to cache them in a stack variable simply because an `**sload`** operation costs 100 gas, when a `mload` costs 3.

### Instances

**File [./contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol) - 11 instances:**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71-L75) [[2]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84) - `stake()`**

[**[3]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113) - `unstake()`**

**[[4] [5]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138-L155) - `rebalanceToWeights()`**

[**[6]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171) - `adjustWeight()`**

[**[7] [8] [9] [10] [11]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-l195) - `addDerivative()`**

### Recommendations

**Cache the storage variables that are being referenced more than once to save an additional 97 gas per additional call.**

## [G-08] - Saving the output of a function when possible to save on gas

This minimizes the number of external calls the functions make.

### Instances

**File [./contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol) - 2 instances:**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71-L75) [[2]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140-L143)**

### Recommendations

Cache the `derivatives[i].balance()`'s value in a stack variable so it does not get called twice.

**This can save upwards of 1000 gas or even more per iteration depending on the derivative’s mechanisms. Typically it will save around 4-5k gas on function call depending on the amount of derivatives.**

## [G-09] - ****Optimize names to save gas****

It is possible to optimize the public/external function names and public member variable names to reduce gas consumption. To see an example of how this optimization works, please [see the example](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9). The interfaces/abstract contracts listed below can be optimized so that the functions that are called frequently consume the least amount of gas during method lookup. By prefixing the method IDs with two zero bytes, up to 128 gas can be saved during deployment, and renaming functions to have lower method IDs can save up to 22 gas per call.

To learn more check the following [article](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).

## [G-10] Emitting an event with a storage variable wastes gas

Emitting an event with a storage variable wastes **130 gas.**

### Instances

**File [./contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol) - 5 instances:**

**[[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L194) - `emit DerivativeAdded(_contractAddress, _weight, derivativeCount);`**

**[[2]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L216) - `emit ChangeMinAmount(minAmount);`**

**[[3]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L225) - `emit ChangeMaxAmount(maxAmount);`**

**[[4]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L234) - `emit StakingPaused(pauseStaking);`**

**[[5]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L243) -**  `**emit UnstakingPaused(pauseUnstaking);**`

### Recommendations

Consider swapping the storage variables out for the stack variables already present in the functions, as most of them already set the storage variable to the stack variable.

## [G-11] [`address.call`](http://address.call) can be optimized with assembly

The standard `(bool success, bytes memory data) = address.call(””)` syntax copies the return data even if we omit the data.

### Instances

**File [./contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol) - 1 instance:**

[**[1]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L124)** 

**File** **[./contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol) - 1 instance:**

[**[2]**](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L110)

**File [./contracts/SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol) - 2 instances:**

**[[3]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L63)**

**[[4]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L76)**

**File `[./contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol)` - 1 instance:**

**[[5]](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L84)**

### Recommendations

Using the following instead of the normal call will save on gas:

```solidity
bool success;                                 
assembly {                                    
        success := call(gas(), toAddress, amount, 0, 0, 0, 0)
}
```