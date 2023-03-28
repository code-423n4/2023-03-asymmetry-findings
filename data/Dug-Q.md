# `Unstaked` event data can be manipulated

When the `unstake()` function is called, `withdraw()` is called on each derivative. At the end of each, the derivative sends its full ether balance to the `SafEth` contract. 

```solidity
(bool sent, ) = address(msg.sender).call{value: address(this).balance}(
    ""
);
```

After receiving ether from each derivative, `unstake()` sends the total received ether to the user and then emits the `Unstaked` event. 

```solidity
emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
```

Because of how this works, a user could send ether to a derivative contract ahead of calling unstake(). This would cause the derivative to send more ether to the `SafEth` contract than it normally would as it would include the amount just sent to the derivative.

The funds would ultimately be returned to the user and the `Unstaked` event would store this manipulated `ethAmountToWithdraw` amount instead of the amount that was actually related to the unstaking of safETH.

As a result, event data and any dependent reporting/analytics would be compromised.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L108-L129

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L107-L114

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L60-L88

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56-L67

# Funds are lost if `stake()` is called before the first derivative added

The `stake()` function will successfully execute even if no derivatives are added. This means that a user could stake safETH and lose all of their funds sent with the call.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L63-L101

A check should be added that causes the call to revert if `mintAmount` is `0`.

# Checks for `minAmount` and `maxAmount` are inconsequential

In code documentation describes the `minAmount` and `maxAmount` parameters as the minimum and maximum amount a user is allowed to stake. However, a user can easily bypass these checks.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L65-L66

Th max amount check is bypassed by simply calling `stake()` more than once.

The min amount check is bypassed by staking the `minAmount` and then calling `unstake()` to withdraw a partial balance.

# Use `Ownable2StepUpgradeable` instead of `OwnableUpgradable` to prevent loss of admin functionality

The `SafeEth` contract currently uses `OwnableUpgradeable` to manage admin functionality. This contract is vulnerable to a loss of admin functionality if `transferOwnership()` is called with the incorrect address. 

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L18

It is recommended to use the `Ownable2StepUpgradeable` contract instead. This contract requires the new owner to accept the ownership transfer before it can be completed. This prevents the loss of functionality if the incorrect address is used.

# Derivative contracts do not emit events for state changes

Generally, it is a good practice to emit events for all state changes. This allows interested parties to easily track changes to the state of the contract.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol

`IDerivative` should include events that are emitted by all derivatives.