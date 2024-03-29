# GAS OPTIMIZATIONS

|             | Issues| Instances|
|:--------:|-------|----------|
| [G-1]   | Use a more recent version of solidity      |4 |
| [G-2]   | Setting the constructor to payable      |4 |
| [G-3]   | OPTIMIZE NAMES TO SAVE GAS      | 37|
| [G-4]   | Don't declare the variables inside the loops costs more gas       |11 |
| [G-5]   | Avoid contract existence checks by using low level calls     | 19|
| [G-6]   |  ADD UNCHECKED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS REQUIRE() OR IF-STATEMENT       |1 |
| [G-7]   | Public function not called by contract declare external to save gas     | 2|
| [G-8]   | With assembly, .call (bool success) transfer can be done gas-optimized      |3 |
| [G-9]   | Empty blocks should be removed or emit something      | 4|
| [G-10]   | Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead      | 2|
| [G-11]   |  Private functions only called once can be inlined to save gas      |1 |
| [G-12]   | Make 3 event parameters indexed when possible      | 5|
| [G-13]   | Save gas with address(0) checks before transfer ownership      | 4|
| [G-14]   | Gas overflow during iteration (DoS)      | 4|
| [G-15]   | Sort Solidity operations using short-circuit mode      | 1|
| [G-16]   | Save gas with the use of the specific import statement      | 4|
| [G-17]   | Instead of calculating the value with keccak256() every time the contract is made pre calculate them before and only give the result to a constant      | 3|

### [G-1] Use a more recent version of solidity

INSTANCES (4):

In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity
2: pragma solidity ^0.8.13;
```
FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
```solidity 
2: pragma solidity ^0.8.13;
```
FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
2: pragma solidity ^0.8.13;
```
FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
2: pragma solidity ^0.8.13;
```
##

### [G-2] Setting the constructor to payable

INSTANCES (4):

APPRIMATE GAS SAVED : 52 GAS 

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity
24:    constructor() { 
```
[WstEth.sol#L24](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L24)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
```solidity 
27:   constructor() {
```
[SfrxEth.sol#L27](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L27)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
38: constructor() {
```
[SafEth.sol#L38](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L38)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
33:    constructor() {
```
[Reth.sol#L33](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L33)

##

### [G-3] OPTIMIZE NAMES TO SAVE GAS

INSTANCES (37):

public/external function names and public member variable names can be optimized to save gas.  Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity  
/// @audit 
initialize(),name(),setMaxSlippage(),withdraw(),deposit(),ethPerDerivative(),balance()
12: contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
```
FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
```solidity 
/// @audit 
initialize(),name(),setMaxSlippage(),withdraw(),deposit(),ethPerDerivative(),balance()
13: contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {

```
FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
/// @audit 
initialize(),stake(),unstake(),rebalanceToWeights(),adjustWeight(),addDerivative(),setMaxSlippage(),setMinAmount(),setMaxAmount(),setPauseStaking(),setPauseUnstaking()
15: contract SafEth is
```
FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
/// @audit 
initialize(),name(),setMaxSlippage(),rethAddress(),swapExactInputSingleHop(),withdraw(),poolCanDeposit(),deposit(),ethPerDerivative(),balance(),poolPrice()
19: contract Reth is IDerivative, Initializable, OwnableUpgradeable {

```
##

### [G-4] Don't declare the variables inside the loops costs more gas

INSTANCES (11):

APPRIMATE GAS SAVED : 99 GAS  

> GAS SAMPLE TEST IN remix ide(Without gas optimizations) :

Before Mitigation :
```solidity
     function testGas() public view {

        for(uint256 i = 0; i < 10; i++) {
          address collateral = msg.sender;
          address  collateral1 = msg.sender;
            
        }
```
The execution Cost : 2176 

After Mitigation :
```solidity
     function testGas() public view {
    address collateral; address  collateral1;
        for(uint256 i = 0; i < 10; i++) {
         collateral = msg.sender;
         collateral1 = msg.sender;
            
        }
```
The execution Cost : 2086 . 

>  Hence clearly after mitigation the gas cost is reduced. Approximately its possible to save 9 gas for every iterations 

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

[SafEth.sol#L71-L96](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71-L96)
[SafEth.sol#L113-L119](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113-L119)
[SafEth.sol#L140-L153](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140-L153)

##

### [G-5] Avoid contract existence checks by using low level calls

INSTANCES (19):

APPRIMATE GAS SAVED : 1900 GAS  


Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

FILE: 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

```solidity
90:    IERC20(_tokenIn).approve(UNISWAP_ROUTER, _amountIn);
101:   amountOut = ISwapRouter(UNISWAP_ROUTER).exactInputSingle(params);
108:   RocketTokenRETHInterface(rethAddress()).burn(amount);
197:   uint256 rethBalance1 = rocketTokenRETH.balanceOf(address(this));
199:   uint256 rethBalance2 = rocketTokenRETH.balanceOf(address(this));
222:   return IERC20(rethAddress()).balanceOf(address(this));
```
[Reth.sol#L90](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L90),[Reth.sol#L101](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L101),[Reth.sol#L108](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L108),[Reth.sol#L197](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L197),[Reth.sol#L199](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L199),[Reth.sol#L222](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L222)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol

```solidity
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

 (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );

   uint256 sfrxBalancePre = IERC20(SFRX_ETH_ADDRESS).balanceOf(
            address(this)
        );

   uint256 sfrxBalancePost = IERC20(SFRX_ETH_ADDRESS).balanceOf(
            address(this)
        );

112: uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
            10 ** 18
        );
115: return ((10 ** 18 * frxAmount) /
            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());

123: return IERC20(SFRX_ETH_ADDRESS).balanceOf(address(this));


```
[SfrxEth.sol#L61-L84](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L61-L84),[SfrxEth.sol#L95-L104](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L95-L104),[SfrxEth.sol#L112-L116](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L112-L116),[SfrxEth.sol#L123](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L123)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
101:  amountOut = ISwapRouter(UNISWAP_ROUTER).exactInputSingle(params);
197:  uint256 rethBalance1 = rocketTokenRETH.balanceOf(address(this));
199:  uint256 rethBalance2 = rocketTokenRETH.balanceOf(address(this));
222:  return IERC20(rethAddress()).balanceOf(address(this));
```
[Reth.sol#L101](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L101),[Reth.sol#L197](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L197),[Reth.sol#L199](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L199),[Reth.sol#L222](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L222)
##

### [G-6] ADD UNCHECKED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS REQUIRE() OR IF-STATEMENT . This saves 30-40 gas

INSTANCES (1):

APPRIMATE GAS SAVED : 40 GAS  

SOLUTION:
```solidity
 require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a } if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
```
This will stop the check for overflow and underflow so it will save gas.

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
200: require(rethBalance2 > rethBalance1, "No rETH was minted");
201: uint256 rethMinted = rethBalance2 - rethBalance1; 
```
[Reth.sol#L200-L201](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L200-L201)

##

### [G-7] Public function not called by contract declare external to save gas 

INSTANCES (2):

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
211: function ethPerDerivative(uint256 _amount) public view returns (uint256) {
221: function balance() public view returns (uint256) {
```
[Reth.sol#L211](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L211),[Reth.sol#L221](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L221)

##

## [G-8] With assembly, .call (bool success) transfer can be done gas-optimized

INSTANCES (3):

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

```solidity

124: (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}("");

```
[SafEth.sol#L124-L126](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L124-L126)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

```solidity
76: (bool sent, ) = WST_ETH.call{value: msg.value}("");
63:  (bool sent, ) = address(msg.sender).call{value: address(this).balance}("");
```
[WstEth.sol#L76](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L76),[WstEth.sol#L63-L65](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L63-L65)

## Recommended Mitigation:

```solidity
bool success;                                 
+ assembly {                                    
+        success := call(gas(), dest, amount, 0, 0)
+          }   
```
##

## [G-9] Empty blocks should be removed or emit something

INSTANCES (4):

Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity
97:  receive() external payable {}
```
[WstEth.sol#L97](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L97)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
```solidity 
126: receive() external payable {}
```
[SfrxEth.sol#L126](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L126)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
246: receive() external payable {}
````
[SafEth.sol#L246](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L246)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
244: receive() external payable {}
```

##

## [G-10] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

INSTANCES (2):

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

<https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html>

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

```solidity
86:  uint24 _poolFee,
240:  (uint160 sqrtPriceX96, , , , , , ) = pool.slot0();
```
[Reth.sol#L86](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L86).[Reth.sol#L240](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L240)

##

## [G-11] Private functions only called once can be inlined to save gas

INSTANCES (1):

APPRIMATE GAS SAVED : 40 GAS  

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol

```solidity

function swapExactInputSingleHop(
        address _tokenIn,
        address _tokenOut,
        uint24 _poolFee,
        uint256 _amountIn,
        uint256 _minOut
    ) private returns (uint256 amountOut) {

```
[Reth.sol#L83-L89](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L83-L89)

##

## [G-12] Make 3 event parameters indexed when possible

INSTANCES (5):

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed

- use 3 event parameters indexed
- If less than 3 all event parameters should be indexed 

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
    event SetMaxSlippage(uint256 indexed index, uint256 slippage);
    event Staked(address indexed recipient, uint ethIn, uint safEthOut);
    event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);
    event WeightChange(uint indexed index, uint weight);
    event DerivativeAdded(
        address indexed contractAddress,
        uint weight,
        uint index
    );
```
##

## [G-13] Save gas with address(0) checks before transfer ownership

INSTANCES (4):

The owner address value is assigned in the constructor, there is no address(0) value control. This means that transfer ownership to ADDRESS(0), The contract must be deployed again. This possibility mean gas consumption.

ADDRESS(0) control is the most error-prone value control . There is the possibility that ownership set to address(0). In this case contract must be redeployed again. This costs the large volume of the gas 

Adding a address(0) control does not cause high gas consumption.

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity
function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
}
```

[WstEth.sol#L33-L36](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L33-L36)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
```solidity
function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
}
```
[SfrxEth.sol#L36-L39](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L36-L39)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
}
```
[Reth.sol#L42-L45](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L42-L45)

### Recommendation:

It is recommended to perform a ADDRESS(0) check for critical OWNER assignments

##

## [G-14] Gas overflow during iteration (DoS)

INSTANCES (4):

Each iteration of the cycle requires a gas flow. A moment may come when more gas is required than it is allocated to record one block. In this case, all iterations of the loop will fail

<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L84>
<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113>
<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140>
<https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L147>

##

## [G-15] Sort Solidity operations using short-circuit mode

INSTANCES (1):

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low, if the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)

```

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

Use ethAmountToRebalance == 0 first then weights[i] == 0 . Because weights[i] is a state variable this consume more gas than ethAmountToRebalance == 0. 

```solidity

148:  if (weights[i] == 0 || ethAmountToRebalance == 0) continue;

```
##

### [G-16] Save gas with the use of the specific import statements

INSTANCES (34):

With the import statement, it saves gas to specifically import only the parts of the contracts, not the complete ones.

Description:

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

### Recommendation:
```solidity
import {contract1 , contract2} from "filename.sol";
``` 

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
```solidity
import "../../interfaces/IDerivative.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../../interfaces/curve/IStEthEthPool.sol";
import "../../interfaces/lido/IWStETH.sol";
```
[WstEth.sol#L4-L8](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L4-L8)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
```solidity
import "../../interfaces/IDerivative.sol";
import "../../interfaces/frax/IsFrxEth.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../../interfaces/curve/IFrxEthEthPool.sol";
import "../../interfaces/frax/IFrxETHMinter.sol";
```
[SfrxEth.sol#L4-L9](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L4-L9)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol
```solidity
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../interfaces/IWETH.sol";
import "../interfaces/uniswap/ISwapRouter.sol";
import "../interfaces/lido/IWStETH.sol";
import "../interfaces/lido/IstETH.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "./SafEthStorage.sol";
import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
```
[SafEth.sol#L4-L11](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L4-L11)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol
```solidity
import "../../interfaces/IDerivative.sol";
import "../../interfaces/frax/IsFrxEth.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../../interfaces/rocketpool/RocketStorageInterface.sol";
import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
import "../../interfaces/IWETH.sol";
import "../../interfaces/uniswap/ISwapRouter.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "../../interfaces/uniswap/IUniswapV3Factory.sol";
import "../../interfaces/uniswap/IUniswapV3Pool.sol";
```
[Reth.sol#L4-L15](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L4-L15)

##

## [G-17] Instead of calculating the value with keccak256() every time the contract is made pre calculate them before and only give the result to a constant

INSTANCES (3):


FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol

```solidity
               keccak256(
                    abi.encodePacked("contract.address", "rocketTokenRETH")
                )
```

[Reth.sol#L69-L71](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L69-L71)

```solidity
                   keccak256(
                        abi.encodePacked("contract.address", "rocketTokenRETH")
                    )
```

[Reth.sol#L190-L192](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L190-L192)

```solidity
                keccak256(
                    abi.encodePacked("contract.address", "rocketTokenRETH")
                )

```
[Reth.sol#L232-L234](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L232-L234)
