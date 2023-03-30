# 1.Use StETHPerToken() function in ethPerDerivative() function in WstEth.sol

## Summary
wstEth contract have another function `stEthPerToken()` that returns the price for exactly 1 ether instead of manually passing the amount in the `getStEthByWstEth(_amount)`
## Vulnerability Details
Here we can see that both the functions have same inner implementation but for the `stEthPerToken()` one ether is hardcoded so we dont need to pass it manually as in `getStEthByWstEth(_amount)`
https://github.com/lidofinance/lido-dao/blob/df95e563445821988baf9869fde64d86c36be55f/contracts/0.6.12/WstETH.sol#L107
https://github.com/lidofinance/lido-dao/blob/df95e563445821988baf9869fde64d86c36be55f/contracts/0.6.12/WstETH.sol#L99

## Impact
When argument is passed in function its gas is determined using its hex form, one non zero byte costs `16 gas` and non zero bytes cost '4 gas'.
Here gas can be saved by just using the function that donot need the argument to be passed in it.
So in total saves arount `400 gas` in total

## Proof of Concept
```solidity
pragma solidity ^0.8.0;

contract test{
    uint amount = 10;
    function test1(uint _amount) public returns(uint)
    {
        return amount;
    }
    function test2() public returns(uint)
    {
        return amount;
    }
}
```
Deployed the above simple code in remix, and passed the argument as 10**18 and on calling first one cosumes `400` less gas than second.

## Tools Used
Manual Review and Remix
## Recommended Mitigation Steps
Use the `stEthPerToken()` function described here
https://github.com/lidofinance/lido-dao/blob/df95e563445821988baf9869fde64d86c36be55f/contracts/0.6.12/WstETH.sol#L107

## 2. Redundant check that does nothing
## Details
Following multiplication and division does nothing as we multiply and divide by same number,
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L215
It can be removed and some gas can be saved as this function calculates price and is called very frequently.
## Mitigation Step
Remove the multiplication and division and simply return the `poolPrice()`

## 3. `Unstake(_safEthAmount)` wastes gas due to lack of check
## Details
Initially we donot check if the user have the necessary balance that the user have passed into function and we go through the loop until we hit the following function and it performs the balance check within it at this line
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L120
## Mitigation
Write necessary check for the balance check in start of the function.
