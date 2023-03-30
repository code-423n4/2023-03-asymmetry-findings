# Findings Summary

| ID     | Title                                                  | Severity |
| ------ | ------------------------------------------------------ | -------- |
| [G-01] | you can use one event instead of two event to save gas | low      |

# [G-01] you can use one event instead of two event to save gas

## Description

there is no need to use 2 event for StakingPaused/UnstakingPaused and/or Staked/Unstaked, you can use one event for pause/unpause and one event for Stake/Unstake and saving some gas.

## context

There are 5 instances of the topic.

```solidity

file: contracts/SafEth/SafEth.sol

 //@audit-info use one event for both
    event StakingPaused(bool indexed paused);
    event UnstakingPaused(bool indexed paused);
    //@audit-info use one event for both below !
     event Staked(address indexed recipient, uint ethIn, uint safEthOut);
    event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L23-L27

## Recommendations

change the events above to something like this :

`event stakeState(bool indexed paused)` and same thing to staked/unstaked event can be set this way.
