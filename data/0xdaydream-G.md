# Details

| Number | Optimization Details | Instances | Total Gas Saved |
|---|---|---|---|
| [G-01] | With assembly, `.call (bool success)` transfer can be done gas-optimized | 5 | 594 |
| [G-02] | Functions guaranteed to revert when callled by normal users can be marked `payable` | 9 | 192 |
| [G-03] | Setting the _constructor_ to `payable` | 4 | - |
| [G-04] | Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | 2 | - |
| [G-05] | Use a more recent version of solidity | 4 | - |
| [G-06] | Optimize names to save gas | - | - |

Total: 24 instances over 6 issues with **786 gas** saved

All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions.


## [G-01] With assembly, `.call (bool success)` transfer can be done gas-optimized

`return` data `(bool success,)` has to be stored due to EVM architecture, but in a usage like below, 'out' and 'outsize' values are given (0,0), this storage disappears and gas optimization is provided.

https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19

5 instances - 4 files:

```diff
contracts/SafEth/SafEth.sol#L124:
-         (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
-             ""
-         );
+        bool sent;                                 
+        address sender = msg.sender;
+        // solhint-disable-next-line
+        assembly {                                    
+            sent := call(gas(), sender, ethAmountToWithdraw, 0, 0, 0, 0)
+        }
```
[SafEth.sol#L124](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L124)

```solidity
contracts/SafEth/derivatives/Reth.sol#L110:
110:        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
111:            ""
112:        );
```
[Reth.sol#L110](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L110)

```solidity
contracts/SafEth/derivatives/SfrxEth.sol#L84:
84:        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
85:            ""
86:        );
```
[SfrxEth.sol#L84](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L84)

```solidity
contracts/SafEth/derivatives/WstEth.sol:
63:        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
64:            ""
65:        );

76:        (bool sent, ) = WST_ETH.call{value: msg.value}("");
```
[WstEth.sol#L63](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L63), [WstEth.sol#L76](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L76)

**Total gas saved: 594**

**Deployment gas saved: 64124**

## [G-02]  Functions guaranteed to revert when callled by normal users can be marked `payable` 

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

9 instances - 1 file:

```solidity
contracts/SafEth/SafEth.sol:
48:    function initialize(
49:        string memory _tokenName,
50:        string memory _tokenSymbol
51:    ) external initializer {

138:    function rebalanceToWeights() external onlyOwner {

165:    function adjustWeight(
166:        uint256 _derivativeIndex,
167:        uint256 _weight
168:    ) external onlyOwner {

182:    function addDerivative(
183:        address _contractAddress,
184:        uint256 _weight
185:    ) external onlyOwner {

202:    function setMaxSlippage(
203:        uint _derivativeIndex,
204:        uint _slippage
205:    ) external onlyOwner {

214:    function setMinAmount(uint256 _minAmount) external onlyOwner {

223:    function setMaxAmount(uint256 _maxAmount) external onlyOwner {

232:    function setPauseStaking(bool _pause) external onlyOwner {

241:    function setPauseUnstaking(bool _pause) external onlyOwner {
```
[SafEth.sol#L48](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L48), [SafEth.sol#L138](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138), [SafEth.sol#L165](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165), [SafEth.sol#L182](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182), [SafEth.sol#L202](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202), [SafEth.sol#L214](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214), [SafEth.sol#L223](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223), [SafEth.sol#L232](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232), [SafEth.sol#L241](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241)

**Recommendation:**
Functions guaranteed to revert when called by normal users can be marked payable  (for ```onlyOwner``` and ```initializer``` functions).

Additionaly, you have to change IDerivative.sol to allow these funcions to be payable

**Total gas saved: 192**

**Deployment gas saved: 25186**

## [G-03] Setting the _constructor_ to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of ```msg.value == 0``` and saves ```13 gas``` on deployment with no security risks.

4 instances - 4 files:

```diff
contracts/SafEth/SafEth.sol#L38:
-     constructor() {
+     constructor() payable {
```
[SafEth.sol#L38](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L38)

```solidity
contracts/SafEth/derivatives/Reth.sol#L33:
33:    constructor() {
```
[Reth.sol#L33](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L33)

```solidity
contracts/SafEth/derivatives/SfrxEth.sol#L27:
27:    constructor() {
```
[SfrxEth.sol#L27](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L27)

```solidity
contracts/SafEth/derivatives/WstEth.sol#L24:
24:    constructor() {
```
[WstEth.sol#L24](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L24)

**Deployment gas saved: 856**

## [G-04] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html 

Use a larger size then downcast where needed.

2 instances - 1 file:

```solidity
contracts/SafEth/derivatives/Reth.sol:
86:        uint24 _poolFee,

240:        (uint160 sqrtPriceX96, , , , , , ) = pool.slot0();
```
[Reth.sol#86](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L86), [Reth.sol#L240](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L240)

## [G-05] Use a more recent version of solidity

In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

4 instances - 4 files:

```solidity
contracts/SafEth/SafEth.sol#L2:
2: pragma solidity ^0.8.13;

contracts/SafEth/derivatives/Reth.sol#L2:
2: pragma solidity ^0.8.13;

contracts/SafEth/derivatives/SfrxEth.sol#L2:
2: pragma solidity ^0.8.13;

contracts/SafEth/derivatives/WstEth.sol#L2:
2: pragma solidity ^0.8.13;
```

## [G-06] Optimize names to save gas
Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92), [here](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) and [here](https://emn178.github.io/online-tools/keccak_256.html).

For example, SafEth.sol function names can be named and sorted according to METHOD ID:

```js
Sighash   |   Function Signature
========================
4cd88b76  =>  initialize(string,string)
3a4b66f1  =>  stake()
2e17de78  =>  unstake(uint256)
d416c096  =>  rebalanceToWeights()
b9c849b6  =>  adjustWeight(uint256,uint256)
a57e3b10  =>  addDerivative(address,uint256)
5e7cc583  =>  setMaxSlippage(uint256,uint256)
897b0637  =>  setMinAmount(uint256)
4fe47f70  =>  setMaxAmount(uint256)
6d49e0b6  =>  setPauseStaking(bool)
d5b7f606  =>  setPauseUnstaking(bool)
```