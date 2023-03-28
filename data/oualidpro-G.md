### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] | Functions guaranteed to revert when called by normal users can be marked payable |16 |
| [G-02] |With assembly, .call (bool success) transfer can be done gas-optimized |5 |
| [G-03] |Setting the constructor to payable |4 |
| [G-04] |Multiple mappings can be combined into a single mapping of an ID to a struct |1 |
| [G-05] |Variable declaration outside the loop to save gas |7|

Total 5 issues

### [G-01] Functions guaranteed to revert when called by normal users can be marked payable (~288 gas)

If a function modifier or require such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are ``CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2)`` which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

In SafeEth.sol file:
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L138
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L168
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L205
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L214
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L223
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L232
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L241


In the Reth.sol file:
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L58
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L107
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L156

In the SfrxEth.sol file:
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L60
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L94

In WstEth.sol file:
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L73

**Recommendation:**
I suggest that you make all functions protected by the onlyOwner modifier ``payable``.


### [G-02] With assembly, ``.call (bool sent)`` transfer can be done gas-optimized (719 gas)

``return`` data ``(bool sent,)`` has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

There are 5 instances of the topic:

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafeEth.sol#L124
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L110
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L84
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L63
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L76


**Recommendation:**
Make the following change in all the previously mentionned files:
```diff
- 124:    (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
- 125:        ""
- 126:    );
+ 123:  bool sent;
+ 124:  assembly {
+ 125:      sent := call(gas(), caller(), ethAmountToWithdraw, 0, 0, 0, 0)
+ 126:  }
```


### [G-03] Setting the constructor to payable (844 gas)

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of ``msg.value == 0`` and saves 13 gas on deployment with no security risks.

There are 5 instances of the topic:
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafeEth.sol#L38
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L33
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L27
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L24

**Recommendation:**
set all the constructors to ``payable``.


### [G-04] Multiple mappings can be combined into a single mapping of an ID to a struct (144 gas)

By combining the following mappings:
File: \contracts\SafEth\SafEthStorage.sol
```solidity
22:     mapping(uint256 => IDerivative) public derivatives; // derivatives in the system
23:     mapping(uint256 => uint256) public weights; // weights for each derivative
```
into a single struct and as most of the time both fields are accessed in the same function, you can save gas for each access due to not having to recalculate the key’s keccak256 hash and that calculation’s associated stack operations.


**Recommendation:**
Make the following change and the required ones in the other files to fit this:
File: \contracts\SafEth\SafEthStorage.sol
```diff
- 22:     mapping(uint256 => IDerivative) public derivatives; // derivatives in the system
- 23:     mapping(uint256 => uint256) public weights; // weights for each derivative
+ 22:     struct DerivStruct {
+ 23:         IDerivative deriv;
+ 24:         uint weight;
+ 25:     }
+ 26:     mapping(uint256 => DerivStruct) public derivatives;
```
**Note:**
This change might be out of scope but it has a good impact on the contracts in the scope.