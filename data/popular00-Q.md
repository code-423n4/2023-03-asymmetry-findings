## Low/Non-Critical
- NC-01 - Ether erroneously sent to the SafEth.sol contract (implementation or proxy) will be locked
- NC-02 - Undesirable treatment of Ethere sent erroneously to the contracts
    - Ether mistakenly sent to the derivative proxy contracts will be sent to the next address to call `SafeEth.sol:unstake()`. At the end of `withdraw()` in each of the derivative contracts, the entire derivative contract balance is sent to the `SafEth` contract. This is then sent to the `msg.sender` that called `unstake()`. 
    - Ether mistakenly sent to the derivative implementation contracts can be retrieved by the _implementation_ `owner` (note that the initialization calls on the implementations are unprotected, and anyone can call `initialize()` to claim `owner` on newly-deployed implementation contracts). 

## QA
- QA-01 - Inconsistency between uint and uint256 usage in `SafEth.sol`
- QA-02 - Inconsistency in variable usage in `Reth.sol:deposit()` - currently using `rethBalance0/1` instead of `rethBalancePre/Post` as is used elsewhere in the project. 
- QA-03 - totalSupply is shadowed (memory variable shadowing function name) in `SafEth.sol:stake()`