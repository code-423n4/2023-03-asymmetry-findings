# GAS OPTIMIZATIONS

##

### [G-1] Use a more recent version of solidity

In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

    2: pragma solidity ^0.8.13;

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
 
    2: pragma solidity ^0.8.13;

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

   2: pragma solidity ^0.8.13;

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol

   2: pragma solidity ^0.8.13;

##

### [G-2] Setting the constructor to payable

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

    24:    constructor() {

[WstEth.sol#L24](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L24)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
 
    27:   constructor() {

[SfrxEth.sol#L27](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L27)

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

   38: constructor() {

[SafEth.sol#L38](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L38)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol

   33:    constructor() {

[Reth.sol#L33](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L33)

##

### [G-3] OPTIMIZE NAMES TO SAVE GAS

public/external function names and public member variable names can be optimized to save gas.  Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol
  
    /// @audit initialize(),name(),setMaxSlippage(),withdraw(),deposit(),ethPerDerivative(),balance()
    12: contract WstEth is IDerivative, Initializable, OwnableUpgradeable {

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/SfrxEth.sol
 
    /// @audit initialize(),name(),setMaxSlippage(),withdraw(),deposit(),ethPerDerivative(),balance()
    13: contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {


FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

    /// @audit 
initialize(),stake(),unstake(),rebalanceToWeights(),adjustWeight(),addDerivative(),setMaxSlippage(),setMinAmount(),setMaxAmount(),setPauseStaking(),setPauseUnstaking()
    15: contract SafEth is


FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol

    /// @audit initialize(),name(),setMaxSlippage(),rethAddress(),swapExactInputSingleHop(),withdraw(),poolCanDeposit(),deposit(),ethPerDerivative(),balance(),poolPrice()
    19: contract Reth is IDerivative, Initializable, OwnableUpgradeable {

##

### [G-4] Don't declare the variables inside the loops costs more gas 

FILE : 2023-03-asymmetry/contracts/SafEth/SafEth.sol

[SafEth.sol#L71-L96](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71-L96)
[SafEth.sol#L113-L119](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113-L119)
[SafEth.sol#L140-L153](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140-L153)

##

### [G-5] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

FILE: 2023-03-asymmetry/contracts/SafEth/derivatives/WstEth.sol

   90:    IERC20(_tokenIn).approve(UNISWAP_ROUTER, _amountIn);
   101:   amountOut = ISwapRouter(UNISWAP_ROUTER).exactInputSingle(params);
   108:   RocketTokenRETHInterface(rethAddress()).burn(amount);
   197:   uint256 rethBalance1 = rocketTokenRETH.balanceOf(address(this));
   199:   uint256 rethBalance2 = rocketTokenRETH.balanceOf(address(this));
   222:   return IERC20(rethAddress()).balanceOf(address(this));

[Reth.sol#L90](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L90),[Reth.sol#L101](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L101),[Reth.sol#L108](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L108),[Reth.sol#L197](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L197),[Reth.sol#L199](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L199),[Reth.sol#L222](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L222)

##

### [G-6] ADD UNCHECKED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS REQUIRE() OR IF-STATEMENT . This saves 30-40 gas

SOLUTION:

 require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a } if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }

This will stop the check for overflow and underflow so it will save gas.

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol

    200: require(rethBalance2 > rethBalance1, "No rETH was minted");
    201: uint256 rethMinted = rethBalance2 - rethBalance1; 

[Reth.sol#L200-L201](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L200-L201)

##

### [G-7] Public function not called by contract declare external to save gas 

FILE : 2023-03-asymmetry/contracts/SafEth/derivatives/Reth.sol

  211: function ethPerDerivative(uint256 _amount) public view returns (uint256) {

  221: function balance() public view returns (uint256) {

[Reth.sol#L211](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L211),[Reth.sol#L221](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L221),




  

  

   








GAS-1	Use selfbalance() instead of address(this).balance	7
GAS-2	Comparing booleans to true or false	2
GAS-3	Use calldata instead of memory for function arguments that do not get mutated	2
GAS-4	Use Custom Errors	10
GAS-5	Don't initialize variables with default value	11
GAS-6	Pre-increments and pre-decrements are cheaper than post-increments and post-decrements	8
GAS-7	Using private rather than public for constants, saves gas	11
GAS-8	Use shift Right/Left instead of division/multiplication if possible	4
GAS-9	Use storage instead of memory for structs/arrays	1
GAS-10	Increments can be unchecked in for-loops	7
GAS-11	Use != 0 instead of > 0 for unsigned integer comparison	1