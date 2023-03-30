## 1. Use storage gaps in all upgradeable contracts

Use storage gaps like all OZ upgradeable contracts avoid the possibility of future storage collisions with mappings and dynamic arrays that are allocated to low numbered storage slots. Although the odds of such a collision are low, it is better to protect against it by reserving storage slots than to expect there are never any problems, due to the severity of the consequences if there is a collision. Some OZ best practices are [already applied](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L36-L37) in other places, so use the [other best practices](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#storage-gaps) to avoid [future storage collisions](https://proxies.yacademy.dev/pages/security-guide/#storage-collision-vulnerability).

## 2. Reentrancy protection improvements for `stake()` or `unstake()`

`stake()` and `unstake()` in SafEth.sol have no reentrancy protection. At a minimum, it is recommended to move [the line with `_burn(msg.sender, _safEthAmount);`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L120) earlier in `unstake()`, because the function calls external contracts in [the for loop](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113-L119) (albeit contracts written by the Assymetry team). Detected with [this slither detector](https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities).

## 3. Use Uniswap TransferHelper.sol `safeApprove()`

Uniswap has a contract intended to help with Uniswap interactions. Use `safeApprove()` [from this contract](https://docs.uniswap.org/contracts/v3/reference/periphery/libraries/TransferHelper#safeapprove) instead of `approve()` before [the Uniswap interaction in Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L90).

## 4. Missing events for critical operations

Several critical operations do not trigger events in `Reth.sol`, `SfrxEth.sol` and `WstEth.sol`. As a result, it is difficult to check the behavior of the contracts. Without events, users and blockchain-monitoring systems cannot easily detect suspicious behavior.

## 5. Incorrect comment

[This comment's description](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L158) is inaccurate and was accidentally copied from a different function.

## 6. No timelock on `addDerivative()` lets owner control user value

`addDerivative()` allows the SafEth contract owner to add ETH derivatives immediately. This can be followed by a call to `rebalanceToWeights()` which will rebalance the position of all existing SafEth holders to hold some of the added derivative. The lack of a timelock causes two problems:

1. The owner could rug the project by creating a fake ETH derivative, adding it to SafEth with `addDerivative()` with a large weight that would cause most of the ETH to be sent to this new derivative, and therefore steal user value.
2. If a new ETH derivative is added with `addDerivative()`, some existing SafEth holders may not want exposure to this new ETH derivative. Using a timelock allows users time to withdraw their position if they do not want this exposure to the new derivative. There is an additional risk that the new derivative contract, the equivalent of Reth.sol, WstEth.sol, or SfrxEth.sol, will have a bug that could put user funds at risk. Using a timelock gives users time to review the new code that will soon control their funds to make sure they trust the new code.