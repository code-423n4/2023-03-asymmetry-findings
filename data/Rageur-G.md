## GAS-1: <X> <= <Y> costs more gas than <X> < <Y> + 1

### Description

In Solididy, the opcode 'less or equal' doesn't exist. So the EVM will translate this by two distinct operation, first the inferior, and then the equal which cost more gas then a strict less.

### Affected file

* Reth.sol (Line: 147, 149)
* SafEth.sol (Line: 65, 66)

## GAS-2: Add unchecked{} for subtractions where the operands cannot underflow because of a previous require() or if statement

### Description

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
 if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
 This will stop the check for overflow and underflow so it will save gas.

### Affected file

* Reth.sol (Line: 201)

## GAS-3: Empty blocks should be removed or emit something

### Description

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

### Affected file

* Reth.sol (Line: 244)
* SafEth.sol (Line: 246)
* SfrxEth.sol (Line: 126)
* WstEth.sol (Line: 97)

## GAS-4: Functions guaranteed to revert when called by normal users can be marked payable

### Description

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

### Affected file

* Reth.sol (Line: 58, 107)
* SafEth.sol (Line: 138, 165, 182, 202, 214, 223, 232, 241)
* SfrxEth.sol (Line: 51, 60)
* WstEth.sol (Line: 48, 56)

## GAS-5: Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

### Description

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

### Affected file

* Reth.sol (Line: 83, 240, 240)