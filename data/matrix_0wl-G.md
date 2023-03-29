# Summary

## Gas Optimizations

|        | Issue                                                                                                                                 |
| ------ | :------------------------------------------------------------------------------------------------------------------------------------ |
| GAS-1  | USE `SELFBALANCE()` INSTEAD OF `ADDRESS(THIS).BALANCE`                                                                                |
| GAS-2  | `<X> += <Y>`/`<X> -= <Y>` COSTS MORE GAS THAN `<X> = <X> + <Y>`/`<X> = <X> - <Y>` FOR STATE VARIABLES                                 |
| GAS-3  | DON’T COMPARE BOOLEAN EXPRESSIONS TO BOOLEAN LITERALS                                                                                 |
| GAS-4  | SETTING THE CONSTRUCTOR TO PAYABLE                                                                                                    |
| GAS-5  | DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION (INSTANCES)                                       |
| GAS-6  | MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT                                                                     |
| GAS-7  | FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE                                                      |
| GAS-8  | OPTIMIZE NAMES TO SAVE GAS                                                                                                            |
| GAS-9  | THE INCREMENT IN FOR LOOP POSTCONDITION CAN BE MADE UNCHECKED                                                                         |
| GAS-10 | PROPER DATA TYPES                                                                                                                     |
| GAS-11 | PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD                                                       |
| GAS-12 | USING SOLIDITY VERSION 0.8.19 WILL PROVIDE AN OVERALL GAS OPTIMIZATION                                                                |
| GAS-13 | ADD `UNCHECKED {}` FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS `REQUIRE()`, `REVERT` OR `IF` STATEMENT |
| GAS-14 | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                                                                                   |
| GAS-15 | USAGE OF `UINT`/`INT` SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD                                                                |
| GAS-16 | USE BYTES32 INSTEAD OF STRING                                                                                                         |

### [GAS-1] USE `SELFBALANCE()` INSTEAD OF `ADDRESS(THIS).BALANCE`

#### Description:

Use selfbalance() instead of address(this).balance when getting your contract’s balance of ETH to save gas.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

111:         uint256 ethAmountBefore = address(this).balance;

121:         uint256 ethAmountAfter = address(this).balance;

139:         uint256 ethAmountBefore = address(this).balance;

144:         uint256 ethAmountAfter = address(this).balance;

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

110:         (bool sent, ) = address(msg.sender).call{value: address(this).balance}(

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

84:         (bool sent, ) = address(msg.sender).call{value: address(this).balance}(

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

63:         (bool sent, ) = address(msg.sender).call{value: address(this).balance}(

```

### [GAS-2] `<X> += <Y>`/`<X> -= <Y>` COSTS MORE GAS THAN `<X> = <X> + <Y>`/`<X> = <X> - <Y>` FOR STATE VARIABLES

#### Description:

Using the addition operator instead of plus-equals saves gas

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

72:             underlyingValue +=

95:             totalStakeValueEth += derivativeReceivedEthValue;

172:             localTotalWeight += weights[i];

192:             localTotalWeight += weights[i];

```

### [GAS-3] DON’T COMPARE BOOLEAN EXPRESSIONS TO BOOLEAN LITERALS

#### Description:

`if (<x> == true) => if (<x>)`, `if (<x> == false) => if (!<x>)`

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

64:         require(pauseStaking == false, "staking is paused");

109:         require(pauseUnstaking == false, "unstaking is paused");

```

### [GAS-4] SETTING THE CONSTRUCTOR TO PAYABLE

#### Description:

Saves ~13 gas per instance

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

38:     constructor() {

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

33:     constructor() {

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

27:     constructor() {

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

24:     constructor() {

```

### [GAS-5] DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION (INSTANCES)

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

66:         require(sent, "Failed to send Ether");

77:         require(sent, "Failed to send Ether");

```

### [GAS-6] MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT

#### Description:

When constants are marked public, extra getter functions are created, increasing the deployment cost. Marking these functions private will decrease gas cost. One can still read these variables through the source code. If they need to be accessed by an external contract, a separate single getter function can be used to return all constants as a tuple.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/derivatives/Reth.sol

20:     address public constant ROCKET_STORAGE_ADDRESS =

22:     address public constant W_ETH_ADDRESS =

24:     address public constant UNISWAP_ROUTER =

26:     address public constant UNI_V3_FACTORY =

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

14:     address public constant SFRX_ETH_ADDRESS =

16:     address public constant FRX_ETH_ADDRESS =

18:     address public constant FRX_ETH_CRV_POOL_ADDRESS =

20:     address public constant FRX_ETH_MINTER_ADDRESS =

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

13:     address public constant WST_ETH =

15:     address public constant LIDO_CRV_POOL =

17:     address public constant STETH_TOKEN =

```

### [GAS-7] FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE

#### Description:

If a function modifier such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

138:     function rebalanceToWeights() external onlyOwner {

168:     ) external onlyOwner {

185:     ) external onlyOwner {

205:     ) external onlyOwner {

214:     function setMinAmount(uint256 _minAmount) external onlyOwner {

223:     function setMaxAmount(uint256 _maxAmount) external onlyOwner {

232:     function setPauseStaking(bool _pause) external onlyOwner {

241:     function setPauseUnstaking(bool _pause) external onlyOwner {

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

107:     function withdraw(uint256 amount) external onlyOwner {

156:     function deposit() external payable onlyOwner returns (uint256) {

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

51:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

60:     function withdraw(uint256 _amount) external onlyOwner {

94:     function deposit() external payable onlyOwner returns (uint256) {

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

48:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

56:     function withdraw(uint256 _amount) external onlyOwner {

73:     function deposit() external payable onlyOwner returns (uint256) {

```

### [GAS-8] OPTIMIZE NAMES TO SAVE GAS

#### Description:

`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. In this report are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).

[Solidity optimize name github](https://github.com/enzosv/solidity-optimize-name)

[Source](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

### [GAS-9] THE INCREMENT IN FOR LOOP POSTCONDITION CAN BE MADE UNCHECKED

#### Description:

This is only relevant if you are using the default solidity checked arithmetic.

The for loop postcondition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body is `2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks.One can manually do this.

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

71:         for (uint i = 0; i < derivativeCount; i++)

84:         for (uint i = 0; i < derivativeCount; i++) {

113:         for (uint256 i = 0; i < derivativeCount; i++) {

140:         for (uint i = 0; i < derivativeCount; i++) {

147:         for (uint i = 0; i < derivativeCount; i++) {

171:         for (uint256 i = 0; i < derivativeCount; i++)

191:         for (uint256 i = 0; i < derivativeCount; i++)

```

### [GAS-10] PROPER DATA TYPES

#### Description:

In Solidity, some data types are more expensive than others. It’s important to be aware of the most efficient type that can be used. Here are a few rules about data types.

Type uint should be used in place of type string whenever possible.

Type uint256 takes less gas to store than uint8

Type bytes should be used over byte[]

If the length of bytes can be limited, use the lowest amount possible from bytes1 to bytes32.

Type bytes32 is cheaper to use than type string and bytes.

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

[Source](https://betterprogramming.pub/how-to-write-smart-contracts-that-optimize-gas-spent-on-ethereum-30b5e9c5db85)

Fixed size variables are always cheaper than dynamic ones.

[Source](https://medium.com/coinmonks/gas-optimization-in-solidity-part-i-variables-9d5775e43dde)

Most of the time it will be better to use a mapping instead of an array because of its cheaper operations.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

26:     event Staked(address indexed recipient, uint ethIn, uint safEthOut);

27:     event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);

28:     event WeightChange(uint indexed index, uint weight);

31:         uint weight,

32:         uint index

71:         for (uint i = 0; i < derivativeCount; i++)

84:         for (uint i = 0; i < derivativeCount; i++) {

92:             uint derivativeReceivedEthValue = (derivative.ethPerDerivative(

140:         for (uint i = 0; i < derivativeCount; i++) {

147:         for (uint i = 0; i < derivativeCount; i++) {

203:         uint _derivativeIndex,

204:         uint _slippage

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

86:         uint24 _poolFee,

240:         (uint160 sqrtPriceX96, , , , , , ) = pool.slot0();

241:         return (sqrtPriceX96 * (uint(sqrtPriceX96)) * (1e18)) >> (96 * 2);

```

### [GAS-11] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD

#### Description:

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/derivatives/Reth.sol

50:     function name() public pure returns (string memory) {

211:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {

221:     function balance() public view returns (uint256) {

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

44:     function name() public pure returns (string memory) {

122:     function balance() public view returns (uint256) {

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

41:     function name() public pure returns (string memory) {

86:     function ethPerDerivative(uint256 _amount) public view returns (uint256) {

93:     function balance() public view returns (uint256) {

```

### [GAS-12] USING SOLIDITY VERSION 0.8.19 WILL PROVIDE AN OVERALL GAS OPTIMIZATION

#### Context:

All contracts

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

2: pragma solidity ^0.8.13;

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

2: pragma solidity ^0.8.13;

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

2: pragma solidity ^0.8.13;

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

2: pragma solidity ^0.8.13;

```

### [GAS-13] ADD `UNCHECKED {}` FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS `REQUIRE()`, `REVERT` OR `IF` STATEMENT

#### Description:

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block [see resource](https://github.com/ethereum/solidity/issues/10695).

`require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }`

This will stop the check for overflow and underflow so it will save gas

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/derivatives/Reth.sol

200:             require(rethBalance2 > rethBalance1, "No rETH was minted");
201:             uint256 rethMinted = rethBalance2 - rethBalance1;

```

### [GAS-14] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

79:     if (totalSupply == 0)
80:            preDepositPrice = 10 ** 18; // initializes with a price of 1
81:     else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

212:        if (poolCanDeposit(_amount))
213:            return
214:                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
215:        else return (poolPrice() * 10 ** 18) / (10 ** 18);

```

### [GAS-15] USAGE OF `UINT`/`INT` SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD

#### Description:

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/derivatives/Reth.sol

86:         uint24 _poolFee,

```

### [GAS-16] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: contracts/SafEth/SafEth.sol

49:         string memory _tokenName,

50:         string memory _tokenSymbol

```

```solidity
File: contracts/SafEth/derivatives/Reth.sol

50:     function name() public pure returns (string memory) {

```

```solidity
File: contracts/SafEth/derivatives/SfrxEth.sol

44:     function name() public pure returns (string memory) {

```

```solidity
File: contracts/SafEth/derivatives/WstEth.sol

41:     function name() public pure returns (string memory) {

```
