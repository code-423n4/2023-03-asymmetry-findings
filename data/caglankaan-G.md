## Using Prefix Operators Costs Less Gas Than Postfix Operators in Loops

### Description
In Solidity, there are two ways to increment a variable within a loop: i++ and ++i. The difference between them is that i++ is a post-increment operator, which increments the value of i after the expression has been evaluated, while ++i is a pre-increment operator, which increments the value of i before the expression has been evaluated.

When using these operators in a loop, it is more gas-efficient to use ++i instead of i++. The reason is that the i++ operator requires an additional SSTORE operation to store the updated value of i, which consumes more gas.

### Code Location 

Applies for all of the for loops
```solidity
for (uint i = 0; i < derivativeCount; i++)
```


### Recommendation
It is recommended to use ++i instead of i++ to increment the value of a uint variable within a loop. This is not applicable outside of loops.

