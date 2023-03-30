- SafEth.sol L75 and L81, L94 and L98 (function stake): don't divide by 10**18 to then multiply by 10**18 (a/10**18 + b/10**18 = (a+b)/10**18)

- SafEth.sol L148 (function rebalanceToWeights): ethAmountToRebalance is not updated in the loop so the check can just be done before the loop to avoid checking the same thing at each iteration.

- SafEth.sol L172 (function adjustWeight): the new total can be computed without looping over each derivative: new total = old total - old weight + new weight 

- SafEth.sol L192 (function addDerivative): the new total can be computed without looping over each derivative: new total = old total + new weight

- SafEth.sol L165 (function adjustWeight) and L182 (function addDerivative): the new weight should be checked to not be 0. As it is the only place where it is modified, this would allow to remove later checks that it is not 0 (L87 and L148) which should be called more frequently thus saving gas for users. If needed, a function "removeDerivative" can be added and should be more efficient by allowing to not loop over derivative which otherwise would have been deactivated by setting their weight to 0.

- Reth.sol L70 (function rethAddress), L190 (function deposit) and L232 (function poolPrice): keccak256(abi.encodePacked("contract.address", "rocketTokenRETH")) can be computed offchain (it never changes).

- Reth.sol L125 (function poolCanDeposit) and L161 (function deposit): keccak256(abi.encodePacked("contract.address", "rocketDepositPool")) can be computed offchain (it never changes).

- Reth.sol L135 (function poolCanDeposit): keccak256(abi.encodePacked("contract.address","rocketDAOProtocolSettingsDeposit")) can be computed offchain (it never changes).

- Reth.sol L215 (function ethPerDerivative): there is no need to divide by 10**18 and then multiply by 10**18 (a*b/b = a)/
