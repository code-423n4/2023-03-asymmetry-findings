## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-01](#GAS-01) | With assembly, `.call (bool sent)` transfer can be done to optimize gas | 5 |
| [GAS-02](#GAS-02) | Setting the constructor to `payable`| 4 |
| [GAS-03](#GAS-03) | Make for loop unchecked | 7 |
| [GAS-04](#GAS-04) | Unnecessary check in `if` condition | 1 |



## [GAS-01]  With assembly, `.call (bool sent)` transfer can be done to optimize gas 

"(bool success, bytes memory returnData) = target.call()" automatically copies the return data to memory even if you omit the returnData variable. 
If a relayer executes transactions with such calls it can lead to a gas-griefing attack.

In the given contracts, it is advisable that `return` data `(bool sent,)` which by default is stored due to EVM architecture, is executed with the parameters 
inOffset, inSize, retOffset, retSize values set to (0,0,0,0). In this way, the storage disappears and gas optimization is provided.

For reference, refer to this thread
https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19


|Contract|Method|Before|After|Gas Saved|
|:-|:-:|:-:|:-:|:-:|
| Reth | deposit | 176462 | 176448 | 14 |
| Reth | withdraw | 181057 | 181003 | 54 |
| SafeEth | rebalanceToWeights | 727618 | 727388 | 230|
| SafeEth | stake | 527253 | 527184 | 69 |
| SafeEth | unstake | 516303 | 516075 | 228|
| SafeEthV2Mock | adminWithdrawDerivative | 195722 | 195668 | 54|
| [Total](#Total) | | | | [649](#649) |


There are 5 instances of the topic.
**Before:**

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

60:        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
```
```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

84:         (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );

```

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

63:        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );

76:        (bool sent, ) = WST_ETH.call{value: msg.value}("");
```

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

63:       (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
            ""
        );

```

**After:**

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

60:     bool sent;
        address addr = address(msg.sender);
        uint256 bal = address(this).balance;
        assembly {
            sent := call(gas(), addr, bal, 0, 0, 0, 0)
        }
```
```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

84:     bool sent;
        address addr = address(msg.sender);
        uint256 bal = address(this).balance;
        assembly {
            sent := call(gas(), addr, bal, 0, 0, 0, 0)
        }

```

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

63:     bool sent;
        address addr = address(msg.sender);
        uint256 bal = address(this).balance;
        assembly {
            sent := call(gas(), addr, bal, 0, 0, 0, 0)
        }


76:      bool sent;
        uint256 bal = msg.value;
        assembly {
            sent := call(gas(), WST_ETH, bal, 0, 0, 0, 0)
        }
```

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

63:     bool sent;
        address addr = address(msg.sender);
        assembly {
            sent := call(gas(), addr, ethAmountToWithdraw, 0, 0, 0, 0)
        }

```

## [GAS-02] Setting the constructor to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. The extra opcodes avoided are 
CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2). 
Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

There are 4 instances of the topic.
**Before:**

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

33:        constructor() {
```

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

27:         constructor() {
```

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

24:        constructor() {
```

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

38:        constructor() {
```

**After:**

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

33:        constructor() payable {
```

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

27:         constructor() payable {
```

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

24:        constructor() payable {
```

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

38:        constructor() payable {
```
## [G-03] Make for loop unchecked

The risk of for loops getting overflowed is extremely low. Because it always increments by 1 and is limited to the arrays length. Even if the arrays are extremely long, 
it will take a massive amount of time and gas to let the for loop overflow.

|Contract|Method|Before|After|Gas Saved|
|:-|:-:|:-:|:-:|:-:|
| SafeEth | addDerivative | 102462 | 102326 | 136|
| SafeEth | adjustWeight | 46519 | 46315 | 204 |
| SafeEth | rebalanceToWeights | 727388 | 726980 | 408 |
| SafeEth | stake | 527184 | 526776 | 308 |
| SafeEth | unstake | 516075 | 515871 | 204|
| [Total](#Total) | | | | [1260](#1260) |

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

71:         for (uint i = 0; i < derivativeCount; i++)

84:         for (uint i = 0; i < derivativeCount; i++) {

113:        for (uint i = 0; i < derivativeCount; i++) {

140:        for (uint i = 0; i < derivativeCount; i++) {

147:        for (uint i = 0; i < derivativeCount; i++) {

171:        for (uint i = 0; i < derivativeCount; i++)

191:        for (uint i = 0; i < derivativeCount; i++)
```
## Recommended Step

```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

+     function unchecked_inc(uint256 x) private pure returns (uint256) {
+          unchecked {
+              return x + 1;
+         }
+      }

-    for (uint i = 0; i < derivativeCount; i++)
+    for (uint i = 0; i < derivativeCount; i = unchecked_inc(i))
```

## [GAS-04] Unnecessary check in `if` condition

The condition `ethAmountToRebalance == 0` is not required because for the given `if` statement to execute, only `weights[i] == 0` has to be only true because if we switch the conditions, 
one of the test's results in an error.
Removing the condition saves **[71](#71)** gas.

**Before:**
```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

L:148      if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
```
**After:**
```solidity
File: 2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

L:148      if (weights[i] == 0) continue;
```