## Potential Impact
This exploit allowed an attacker to “double exercise” ETH value and steal the collateral posted by certain users.

## Proof of Concept
1. In `SafEth.sol` Contract allows anyone to stake their ETH.
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L63-L101

2. However inside the `stake()` function there is `msg.value` used inside a loop.
```Solidity
for (uint i = 0; i < derivativeCount; i++) {
            uint256 weight = weights[i];
            IDerivative derivative = derivatives[i];
            if (weight == 0) continue;
            uint256 ethAmount = (msg.value * weight) / totalWeight;

            // This is slightly less than ethAmount because slippage
            uint256 depositAmount = derivative.deposit{value: ethAmount}();
            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
                depositAmount
            ) * depositAmount) / 10 ** 18;
            totalStakeValueEth += derivativeReceivedEthValue;
        }
```
```Solidity
uint256 ethAmount = (msg.value * weight) / totalWeight;
```
3. As shown in the above code snippet, the loop function getting total amount of derivatives worth of ETH in system

4. This leads to the same `msg.value` amount of ETH would be re-used in the second or further `stake()` calls in the same transaction. But, only the first batch of ETH is received.

References(Past Incident):
- https://peckshield.medium.com/opyn-hacks-root-cause-analysis-c65f3fe249db
- https://blog.trailofbits.com/2021/12/16/detecting-miso-and-opyns-msg-value-reuse-vulnerability-with-slither/
- https://medium.com/opyn/opyn-eth-put-exploit-c5565c528ad2

## Tools Used

Slither

## Recommended Mitigation Steps

1. Use a local variable `msgValue` to keep the amount of `msg.value` in Solidity. This allows to calculate and book-keep how much ETH had been staked. 
2. In addition, `address(this).balance` could be used to check if the smart contract does have enough ETH as indicated by `msg.value`.