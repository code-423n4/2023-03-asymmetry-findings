#  L-01 Missing events for critical changes

## Impact

This would lead to lack of track keeping of the status of the system off-chain 

## Proof of Concept

[WsEth.setMaxSlippage()](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48-L50)

[SfrxEth.setMaxSlippage()](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53)

[SafEth.setMaxSlippage()](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202-L208)

[Reth.setMaxSlippage()](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L58-L60)

All four contracts have  the  `setMaxSlippage()` function but an event is only emitted when a call is made on SafEth.sol, as i understand this is due to the fact that  SafEth.sol is the main contract for staking and unstaking nonetheless, I’d recommend an event is emitted in all four contracts check this and if that's not the case, include it in the natspec comments



## Tool used

Manual Review

## Recommendation

Consider emitting events when these addresses/values are updated. This will be more transparent and it will make it easier to keep track of the status of the system.
