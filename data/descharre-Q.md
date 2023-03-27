# Summary
## Low Risk
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|1       |Use two-step ownership pattern|1|
|2       |Eth will be locked when calling `stake()` when all weights are 0 |1|
|3       |Slippage can be set to 0 |1|
|4       |Owner can renounce Ownership |1|
|5       |Eth send with `receive()` will be locked  |1|


## Non critical
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|1       | Unspecific compiler version pragma | 1 |
|2       |Use scientific notation (1E18) rather than exponentation (10**18)|14 |
|3       |Wrong natspec comments|1 |

# Details
## Low Risk 
## [L-01] Use two-step ownership pattern
[SafEth](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L9) uses Openzeppelin `OwnableUpgradeable` contract. However with this implementation it's possible that the onlyOwner role mistakenly transfers ownership to a wrong address, resulting in a loss of the onlyOwner role. Consider using the Openzeppelin [`Ownable2StepUpgradeable`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol) contract.
## [L-02] Eth will be locked when calling `stake()` when all weights are 0
When the owner sets all the weights to 0 by accident and a user calls [`stake()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63-L101), the mintAmount will be 0 and the transaction will succeed. This means the ether will still be send to the contract but no derivatives will be minted. Because no derivates and SafEth is minted, the ether will be locked in the contract forever.

This can be prevented by adding a check in the [`adjustWeight()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175) function where you check if totalWeight is more than 0.

## [L-03] Slippage can be set to 0
There is no minimum on the slippage when the owner calls the [`setMaxSlippage()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202-L208) function. This means when the slippage is set to 0 it might lead to a lot of failed transactions because there will be no price change tolerated. This can be solved by adding a minimum amount that the owner can set the slippage too.

## [L-04] Owner can renounce Ownership
Because the protocol uses Openzeppelin `OwnableUpgradeable` contract, the owner can renounce his ownership. When this happens, the protocol will be unupgradeable. A mitigation is to override the function with an empty function.
## [L-05] Eth send with `receive()` will be locked 
When a user calls the `receive()` function with some eth, all the eth will be locked in the contract forever. This is because [`unstake()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108-L129) and [`rebalanceToWeights()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138-L154) has the variables ethAmountBefore and ethAmountAfter and it only send the difference between them to the user. 

There are three mitigations for this:
- remove the payable modifier
- send the whole contract balance to the user when he unstakes
- have a withdraw function.

This also counts for the [`stake()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63-L101) function, a small portion of the deposited ether will be held in the contract and locked. 
## Non critical
## [N-01] Unspecific compiler version pragma
Avoid floating pragmas for non-library contracts.

Contracts should be deployed with the same compiler version and flags that they have been tested the most with. Locking the pragma helps ensure that contracts do not accidentally get deployed using, for example, the latest compiler which may have higher risks of undiscovered bugs. Contracts may also be deployed by others and the pragma indicates the compiler version intended by the original authors.

## [N-02] Use scientific notation (1E18) rather than exponentation (10**18)
Scientific notation should be used for better code readability.
- [SafEth.sol#L54-L55](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L54-L55)
- [SafEth.sol#L75](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L75)
- [SafEth.sol#L80-L81](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L80-L81)
- [SafEth.sol#L94](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L94)
- [SafEth.sol#L98](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L98)
- [Reth.sol#L44](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L44)
- [Reth.sol#L171-L174](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L171-L174)
- [Reth.sol#L214-L215](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L214-L215)
- [SfrxEth.sol#L38](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L38)
- [SfrxEth.sol#L74-L75](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75)
- [SfrxEth.sol#L113-L115](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L113-L115)
- [WstEth.sol#L35](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L35)
- [WstEth.sol#L60](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L60)
- [WstEth.sol#L87](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L87)
## [N-03] Wrong natspec comments
The natspec comment of the function [`adjustWeight()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L158) says Adds new derivative to the index fund. But this is for the function `addDerivative()`.