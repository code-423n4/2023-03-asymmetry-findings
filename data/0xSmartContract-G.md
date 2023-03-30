### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] | With assembly, `.call (bool success)` transfer can be done gas-optimized |5 |
| [G-02] |Avoid using ``state variable`` in emit| 4 |
| [G-03] |Compute known value off-chain |4 |
| [G-04] |Empty blocks should be removed or emit something |4 |
| [G-05] |Add ``unchecked {}`` for subtractions where the operands cannot underflow because of a previous ``require`` or ``if`` statement |1|
| [G-06] |String literals passed to abi.encode()/abi.encodePacked() should not be split by commas  |4 |
| [G-07] |++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops | 7 |
| [G-08] | Change ``public`` function visibility to ``external`` | 8 |
| [G-09] |Use a more recent version of solidity  | 4 |
| [G-10] | Sort Solidity operations using short-circuit mode  |1 |
| [G-11] | Setting the _constructor_ to `payable`|4 |
| [G-12] |Functions guaranteed to revert_ when callled by normal users can be marked `payable`  |17 |
| [G-13] |Optimize names to save gas | All contracts |

Total 13 issues


### [G-01] With assembly, `.call (bool success)` transfer can be done gas-optimized

`return` data `(bool success,)` has to be stored due to EVM architecture, but in a usage like below, 'out' and 'outsize' values are given (0,0), this storage disappears and gas optimization is provided.

https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19

There are 5 instances of the topic.

```diff
contracts\SafEth\SafEth.sol:

  123          // solhint-disable-next-line
- 124:         (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
  125:             ""
  126:         );
+              address addr = payable(msg.sender);  
+              bool sent;
+              assembly {
+                 sent := call(gas(), addr, ethAmountToWithdraw, 0, 0, 0, 0)
+              }  
  127:         require(sent, "Failed to send Ether");
  128          emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L124-L126


```solidity
contracts\SafEth\derivatives\Reth.sol:

  110:         (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
  111:             ""
  112:         );
  113:         require(sent, "Failed to send Ether");
  114:     }

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L110-L113


```solidity
contracts\SafEth\derivatives\SfrxEth.sol:

  83:         // solhint-disable-next-line
  84:         (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
  85:             ""
  86:         );
  87:         require(sent, "Failed to send Ether");
  88:     }

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L84-L88


```solidity
contracts\SafEth\derivatives\WstEth.sol:
  
  62:         // solhint-disable-next-line
  63:         (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
  64:             ""
  65:         );
  66:         require(sent, "Failed to send Ether");


  
  75:         // solhint-disable-next-line
  76:         (bool sent, ) = WST_ETH.call{value: msg.value}("");
  77:         require(sent, "Failed to send Ether");

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L63-L66

### [G-02] Avoid using ``state variable`` in emit

Using the current memory value here will save about 100 gas.

100 gas *4  = ~ 400 gas

4 results - 1 file:
```solidity
contracts\SafEth\SafEth.sol:
  
  214:     function setMinAmount(uint256 _minAmount) external onlyOwner {
  215:         minAmount = _minAmount;
  216:         emit ChangeMinAmount(minAmount);
  217:     }

  223:     function setMaxAmount(uint256 _maxAmount) external onlyOwner {
  224:         maxAmount = _maxAmount;
  225:         emit ChangeMaxAmount(maxAmount);
  226:     }

  232:     function setPauseStaking(bool _pause) external onlyOwner {
  233:         pauseStaking = _pause;
  234:         emit StakingPaused(pauseStaking);
  235:     }

  241:     function setPauseUnstaking(bool _pause) external onlyOwner {
  242:         pauseUnstaking = _pause;
  243:         emit UnstakingPaused(pauseUnstaking);
  244:     }

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214-L217


```diff
contracts\SafEth\SafEth.sol:
  
  214:     function setMinAmount(uint256 _minAmount) external onlyOwner {
- 215:         minAmount = _minAmount;
- 216:         emit ChangeMinAmount(minAmount);
+ 216:         emit ChangeMinAmount(_minAmount);
+ 215:         minAmount = _minAmount;
  217:     }

```


### [G-03] Compute known value off-chain

If it is known which data to hash, no more computational power consumption is needed to hash using keccak256, it causes 2 times more gas consumption.

4 results - 1 file:
```solidity
contracts\SafEth\derivatives\Reth.sol:
   68              RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
   69:                 keccak256(
   70:                     abi.encodePacked("contract.address", "rocketTokenRETH")
   71                  )

  123          ).getAddress(
  124:                 keccak256(
  125:                     abi.encodePacked("contract.address", "rocketDepositPool")
  126                  )

  160          ).getAddress(
  161:                 keccak256(
  162:                     abi.encodePacked("contract.address", "rocketDepositPool")
  163                  )

  231          ).getAddress(
  232:                 keccak256(
  233:                     abi.encodePacked("contract.address", "rocketTokenRETH")
  234                  )

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L69-L72


**Recommendation Code:**
*contracts\SafEth\derivatives\Reth.sol:*
```diff
         // keccak256(abi.encodePacked("contract.address", "rocketTokenRETH"))
+        bytes32 internal constant  ROCKET_TOKEN_RETH= 0xe3744443225bff7cc22028be036b80de58057d65a3fdca0a3df329f525e31ccc;

         //keccak256(abi.encodePacked("contract.address", "rocketDepositPool"))
+        bytes32 internal constant ROCKET_DEPOSIT_POOL = 0x65dd923ddfc8d8ae6088f80077201d2403cbd565f0ba25e09841e2799ec90bb2;

  66:     function rethAddress() private view returns (address) {
  67:         return
- 68:             RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
- 69:                 keccak256(
- 70:                     abi.encodePacked("contract.address", "rocketTokenRETH")
- 71:                 )
+                RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(ROCKET_TOKEN_RETH
  72:             );
  73:     }

```

### [G-04] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}). Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.


4 results - 4 files:
```solidity
contracts\SafEth\SafEth.sol:
  
  246:     receive() external payable {}

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L244


```solidity
contracts\SafEth\derivatives\Reth.sol:
 
  244:     receive() external payable {}

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L246


```solidity
contracts\SafEth\derivatives\SfrxEth.sol:
 
  126:     receive() external payable {}

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L126


```solidity
contracts\SafEth\derivatives\WstEth.sol:
 
  97:     receive() external payable {}

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L97

### [G-05] Add ``unchecked {}`` for subtractions where the operands cannot underflow because of a previous ``require`` or ``if`` statement

```
require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a } 
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
```
This will stop the check for overflow and underflow so it will save gas.

1 result - 1 file:

```diff
contracts\SafEth\derivatives\Reth.sol:

  156:     function deposit() external payable onlyOwner returns (uint256) {

  197:             uint256 rethBalance1 = rocketTokenRETH.balanceOf(address(this));
  198:             rocketDepositPool.deposit{value: msg.value}();
  199:             uint256 rethBalance2 = rocketTokenRETH.balanceOf(address(this));
  200:             require(rethBalance2 > rethBalance1, "No rETH was minted");
+                  unchecked {  
  201:                 uint256 rethMinted = rethBalance2 - rethBalance1;
  202:                 return (rethMinted);
+                  }   
  203:         }

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L200-L202


### [G-06] String literals passed to abi.encode()/abi.encodePacked() should not be split by commas

String literals can be split into multiple parts and still be considered as a single string literal. Adding commas between each chunk makes it no longer a single string, and instead multiple strings. EACH new comma costs 21 gas due to stack operations and separate MSTOREs

4 results - 1 file:
```solidity
contracts\SafEth\derivatives\Reth.sol:

   69:                 keccak256(
   70:                     abi.encodePacked("contract.address", "rocketTokenRETH")
   71                  )


  124:                 keccak256(
  125:                     abi.encodePacked("contract.address", "rocketDepositPool")
  126                  )


  161:                 keccak256(
  162:                     abi.encodePacked("contract.address", "rocketDepositPool")
  163                  )


  232:                 keccak256(
  233:                     abi.encodePacked("contract.address", "rocketTokenRETH")
  234                  )

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L69-L72

### [G-07] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops

In the use of for loops, this structure, which will reduce gas costs. This saves 30-40 gas per loop.

7 results - 1 file:
```solidity
contracts\SafEth\SafEth.sol:

   71:    for (uint i = 0; i < derivativeCount; i++)

   84:    for (uint i = 0; i < derivativeCount; i++) {

  113:    for (uint256 i = 0; i < derivativeCount; i++) {

  140:    for (uint i = 0; i < derivativeCount; i++) {

  147:    for (uint i = 0; i < derivativeCount; i++) {

  171:    for (uint256 i = 0; i < derivativeCount; i++)

  191:    for (uint256 i = 0; i < derivativeCount; i++)

```

**Recommendation Code:**
```diff
- 71:    for (uint i = 0; i < derivativeCount; i++)
+ 71:    for (uint i = 0; i < derivativeCount; i = unchecked_inc(i))


+ function unchecked_inc(uint i) internal pure returns (uint) {
+       unchecked {
+           return i + 1;
+        }
+    }

```

### [G-08] Change ``public`` function visibility to ``external``

Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

8 results - 3 files:
```solidity
contracts\SafEth\derivatives\Reth.sol:

   50:     function name() public pure returns (string memory) {

  211:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {

  221:     function balance() public view returns (uint256) {

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L50


```solidity
contracts\SafEth\derivatives\SfrxEth.sol:

   44:     function name() public pure returns (string memory) {

  122:     function balance() public view returns (uint256) {

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L44


```solidity
contracts\SafEth\derivatives\WstEth.sol:

  41:     function name() public pure returns (string memory) {

  86:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {
  
  93:     function balance() public view returns (uint256) {

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L41


### [G-09] Use a more recent version of solidity

> Solidity 0.8.10 has a useful change that reduced gas costs of external calls which expect a return value. 
> 
> In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.
> 
> In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.
> 
> In 0.8.18 Allow replacing the previously hard-coded cleanup sequence by specifying custom steps after a colon delimiter (:) in the sequence string. Eliminate keccak256 calls if the value was already calculated by a previous call and can be reused.

4 results - 4 files:
```solidity
contracts\SafEth\SafEth.sol:

  2: pragma solidity ^0.8.13;

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2


```solidity 
contracts\SafEth\derivatives\Reth.sol:

  2: pragma solidity ^0.8.13;

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2

```solidity
contracts\SafEth\derivatives\SfrxEth.sol:

  2: pragma solidity ^0.8.13;

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2

```solidity
contracts\SafEth\derivatives\WstEth.sol:

  2: pragma solidity ^0.8.13; 

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2


### [G-10] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses ```OR/AND``` logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```
1 result - 1 file:
```solidity
contracts\SafEth\SafEth.sol:

  148:             if (weights[i] == 0 || ethAmountToRebalance == 0) continue;

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L148


### [G-11] Setting the _constructor_ to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of ```msg.value == 0``` and saves ```13 gas``` on deployment with no security risks.

4 results - 4 files:
```solidty
contracts\SafEth\SafEth.sol:

  38:     constructor() {

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L38

```solidity
contracts\SafEth\derivatives\Reth.sol:

  33:     constructor() {

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L33

```solidity
contracts\SafEth\derivatives\SfrxEth.sol:

  27:     constructor() {

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L27

```solidity
contracts\SafEth\derivatives\WstEth.sol:

  24:     constructor() {

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L24


**Recommendation:**
Set the constructor to ```payable```


### [G-12]  Functions guaranteed to revert_ when callled by normal users can be marked `payable` 

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

17 results - 4 files:
```solidity
contracts\SafEth\SafEth.sol:

  138:    function rebalanceToWeights() external onlyOwner {

  165     function adjustWeight(
  166         uint256 _derivativeIndex,
  167         uint256 _weight
  168:     ) external onlyOwner {

  182     function addDerivative(
  183         address _contractAddress,
  184         uint256 _weight
  185:     ) external onlyOwner {

  202     function setMaxSlippage(
  203         uint _derivativeIndex,
  204         uint _slippage
  205:     ) external onlyOwner {

  214:    function setMinAmount(uint256 _minAmount) external onlyOwner {

  223:    function setMaxAmount(uint256 _maxAmount) external onlyOwner {

  232:    function setPauseStaking(bool _pause) external onlyOwner {

  241:    function setPauseUnstaking(bool _pause) external onlyOwner {

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138


```solidity
contracts\SafEth\derivatives\Reth.sol:

   58:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

  107:    function withdraw(uint256 amount) external onlyOwner {

  156:    function deposit() external payable onlyOwner returns (uint256) {

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58


```solidity
contracts\SafEth\derivatives\SfrxEth.sol:

  51:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

  60:     function withdraw(uint256 _amount) external onlyOwner {

  94:     function deposit() external payable onlyOwner returns (uint256) {

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51


```solidity
contracts\SafEth\derivatives\WstEth.sol:

  48:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

  56:     function withdraw(uint256 _amount) external onlyOwner {

  73:     function deposit() external payable onlyOwner returns (uint256) {

```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48


**Recommendation:**
Functions guaranteed to revert when called by normal users can be marked payable  (for only ```onlyOwner``` functions)

### [G-13] Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via ```Method ID```. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because ```22 gas``` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

All Contracts

SafEthl.sol function names can be named and sorted according to METHOD ID

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