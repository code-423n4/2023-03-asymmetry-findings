### Gas Optimizations List

| Number | Optimization Details                               | Instances |
| :----: | :------------------------------------------------- | :-------: |
| [G-01] | Use scientific notation                            |    18     |
| [G-02] | Unnecessary computation                            |     3     |
| [G-03] | Mark functions as payable                          |    17     |
| [G-04] | Change function visibility from public to external |     2     |
| [G-05] | Use short circuiting to save gas                   |     1     |
| [G-06] | Remove unnecessary variables                       |     5     |

Total 6 issues

#### [G-01] Use scientific notation

Use scientific notation like `1e18` rather than `10 ** 18` to avoid an unnecessary arithmetic operation and save gas.

```solidity
File: contracts/SafEth/SafEth.sol

54:    minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum

55:    maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum

80:    preDepositPrice = 10 ** 18; // initializes with a price of 1

81:    else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

94:    ) * depositAmount) / 10 ** 18;

98:    uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```

```solidity
File: contracts/SafEth/deriviatives/Reth.sol

44:     maxSlippage = (1 * 10 ** 16); // 1%

171:    uint rethPerEth = (10 ** 36) / poolPrice();

173:    uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) * ((10 ** 18 - maxSlippage))) / 10 ** 18);

214:    RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);

215:    else return (poolPrice() * 10 ** 18) / (10 ** 18);
```

```solidity
File: contracts/SafEth/deriviatives/SfrxEth.sol

38:     maxSlippage = (1 * 10 ** 16); // 1%

74:    uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) * (10 ** 18 - maxSlippage)) / 10 ** 18;

112:    uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(10 ** 18);

115:    return ((10 ** 18 * frxAmount) /
```

```solidity
File: contracts/SafEth/deriviatives/WstEth.sol

35:    maxSlippage = (1 * 10 ** 16); // 1%

60:    uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;

87:    return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
```

#### [G-02] Unnecessary computation

1. Reorder code to eliminate unnecessary computation.

   ```solidity
   File: contracts/SafEth/SafEth.sol

   // before - retrieving value and creating an IDerivative
   // is unnecessary if weight is zero
   86:    IDerivative derivative = derivatives[i];
   87:    if (weight == 0) continue;

   // after - an IDerivative is only created when necessary
   // (when weight > 0)
   86:    if (weight == 0) continue;
   87:    IDerivative derivative = derivatives[i];
   ```

1. Assign the value of a storage variable to a local one when possible to avoid multiple reads.

   ```solidity
   File: contracts/SafEth/SafEth.sol

   // before - potentially calling derivatives[i].balance() twice per loop
   140:    for (uint i = 0; i < derivativeCount; i++) {
   141:        if (derivatives[i].balance() > 0)
   142:            derivatives[i].withdraw(derivatives[i].balance());
   143:    }

   // after - save the result of derivatives[i].balance() to a local
   // variable to reduce unnececssary storage reads/function calls
   140:    for (uint i = 0; i < derivativeCount; i++) {
   141:        uint256 balance = derivatives[i].balance();
   142:        if (balance > 0)
   143:            derivatives[i].withdraw(balance);
   144:    }
   ```

1. Assign the value of a storage variable to a local one when possible to avoid multiple reads.

   ```solidity
   File: contracts/SafEth/SafEth.sol

   // before - potentially accessing weights[i] twice per loop
   147:    for (uint i = 0; i < derivativeCount; i++) {
   148:        if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
   149:        uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
   150:            totalWeight;

   // after - save weights[i] to a local variable to reduce
   // unnececssary storage reads
   147:    for (uint i = 0; i < derivativeCount; i++) {
   148:        uint256 weight = weights[i];
   149:        if (weight == 0 || ethAmountToRebalance == 0) continue;
   150:        uint256 ethAmount = (ethAmountToRebalance * weight) /
   151:            totalWeight;
   ```

#### [G-03] Mark functions as payable

Functions guaranteed to revert when called by normal users can be marked `payable`. If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

1. [SafEth.sol Line 138](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138) `rebalanceToWeights()`
2. [SafEth.sol Line 165](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165) `adjustWeight()`
3. [SafEth.sol Line 182](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182) `addDerivative()`
4. [SafEth.sol Line 202](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202) `setMaxSlippage()`
5. [SafEth.sol Line 214](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214) `setMinAmount()`
6. [SafEth.sol Line 223](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223) `setMaxAmount()`
7. [SafEth.sol Line 232](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232) `setPauseStaking()`
8. [SafEth.sol Line 241](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241) `setPauseUnstaking()`
9. [Reth.sol Line 58](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58) `setMaxSlippage()`
10. [Reth.sol Line 107](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107) `withdraw()`
11. [Reth.sol Line 156](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L156) `deposit()`
12. [SfrxEth.sol Line 51](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51) `setMaxSlippage()`
13. [SfrxEth.sol Line 60](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60) `withdraw()`
14. [SfrxEth.sol Line 94](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L94) `deposit()`
15. [WstEth.sol Line 48](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48) `setMaxSlippage()`
16. [WstEth.sol Line 56](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56) `withdraw()`
17. [WstEth.sol Line 73](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L73) `deposit()`

#### [G-04] Change function visibility from public to external

Change function visibility from `public` to `external` if the function is never called from within the contract. For public functions, the input parameters are copied to memory automatically, which costs gas. If the function is only called externally, then it should be marked as external since the parameters for an external function are not copied into memory but are read from calldata directly.

1. [Reth.sol Line 211](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L211) `ethPerDerivative()`
2. [WstEth.sol Line 86](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86) `ethPerDerivative()`

#### [G-05] Use short circuiting to save gas

Use short circuiting to save gas by putting the cheaper comparison first.

```solidity
File: contracts/SafEth/SafEth.sol

// before - always reading storage variable
148:    if (weights[i] == 0 || ethAmountToRebalance == 0) continue;

// after - don't have to read storage variable if local variable is zero
148:    if (ethAmountToRebalance == 0 || weights[i] == 0) continue;
```

#### [G-06] Remove unnecessary variables

Removing unnecessary variables can make the code simpler and save some gas.

1. [SafEth.sol Line 121](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#121)

   ```solidity
   File: contracts/SafEth/SafEth.sol

   // before - using unnecessary variable ethAmountAfter
   121:    uint256 ethAmountAfter = address(this).balance;
   122:    uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;

   // after - using address balance directly
   // (also using selfbalance() rather than address(this).balance)
   122:    uint256 ethAmountToWithdraw = selfbalance() - ethAmountBefore;
   ```

1. [SafEth.sol Line 144](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#144)

   ```solidity
   File: contracts/SafEth/SafEth.sol

   // before - using unnecessary variable ethAmountAfter
   144:    uint256 ethAmountAfter = address(this).balance;
   145:    uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;

   // after - using address balance directly
   // (also using selfbalance() rather than address(this).balance)
   145:    uint256 ethAmountToRebalance = selfbalance() - ethAmountBefore;
   ```

1. [Reth.sol Line 201](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L201)

   ```solidity
   File: contracts/SafEth/derivatives/Reth.sol

   // before - using unnecessary variable rethMinted
   201:    uint256 rethMinted = rethBalance2 - rethBalance1;
   202:    return (rethMinted);

   // after - returning difference value directly
   201:    return (rethBalance2 - rethBalance1);
   ```

1. [SfrxEth.sol Line 89](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L89)

   ```solidity
   File: contracts/SafEth/derivatives/SfrxEth.sol

   // before - using unnecessary variable sfrxBalancePost
   89:    uint256 sfrxBalancePost = IERC20(SFRX_ETH_ADDRESS).balanceOf(address(this));
   90:    return sfrxBalancePost - sfrxBalancePre;

   // after - returning difference value directly
   90:    return (IERC20(SFRX_ETH_ADDRESS).balanceOf(address(this)) - sfrxBalancePre);
   ```

1. [WstEth.sol Line 77](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L77)

   ```solidity
   File: contracts/SafEth/derivatives/WstEth.sol

   // before - using unnecessary variables wstEthBalancePost and wstEthAmount
   77:    uint256 wstEthBalancePost = IWStETH(WST_ETH).balanceOf(address(this));
   78:    uint256 wstEthAmount = wstEthBalancePost - wstEthBalancePre;
   79:    return (wstEthAmount);

   // after - returning difference value directly
   79:    return (IWStETH(WST_ETH).balanceOf(address(this)) - wstEthBalancePre);
   ```
