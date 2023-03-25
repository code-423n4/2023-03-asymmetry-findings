## change `i++` to `++i` can save gas

In code https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71、https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84、https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113、https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140

in for loop ++i is more gas-saving than i++, recommend change i++ to ++i. Even you can use unchecked{ ++i } to save more gas.

## using unchecked{} identifiers to save gas

In code https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L188 ;
`derivativeCount++` can change to 
```solidity
unchecked{
    derivativeCount++;
}
```
Because derivativeCount should never overflow or underflow.
