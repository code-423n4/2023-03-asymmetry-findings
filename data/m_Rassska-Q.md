
# Overall Analysis
* ## About the Protocol
    > [Asymmetry Finance](https://www.asymmetry.finance/) provides an opportunity for stakers to diversify their staked eth across many liquid staking derivatives. It's not a doubt that the Lido has about 80% of the liquid staking market and Asymmetry Finance introduces a great solution to make the LSM more decentralized. 

</br>

* ## Test Coverage and Code Quality
    * The research made over the scope provided by Asymmetry Finance confirms that the quality of the code is on pretty decent level. However, lack of the documentation makes it a little harder to understand the overall architecture behind Asymmetry Finance. Fortunately, the team took some efforts towards documenting the NatSpec for almost all the functions within a scope. For further code readability improvements, consider utilizing: modifiers, named return values. Although, it slightly increases the bytecode size, but it provides an intrinsic gas consumption optimization along with a great readability.

    * The test coverage is well-defined, thus it covers the crucial logic in a very solid way. Also, it gives a huge help in order to follow the main components, like `stake()` and `unstake()` for auditors. The recommendations towards utilizing fuzzing tools or Certora Prover are relevant for protocol's safety against potential edge cases with a significant impact missed after the manual review. 
    
</br>

* ## Centralization Risks
  * The current implementation allows for the SafEth Owner to sweep all the underlying ETH by adding malicious derivative, adjusting the 100% weight to that and then invoking `rebalanceToWeights()`. As a result of that, 100% of user's assets are lost. Anyways, there is a risk of the wallet being compromised and the attacker could simply perform the following attack. In order to mitigate this, introduce a timelock upon adding new derivative, so that users could potentially save their underlying eth.
  
  * Since derivatives are also upgradeable contracts, it is possible for proxy admins to replace the implementation with a malicious one and then destroy the proxy stealing all the funds held by a certain derivative. In order to prevent such risks, introduce the multi-sig wallet as an Owner for Safeth.

</br>

* ## Re-entrancy protection
  * Currently, `stake()` and `unstake()` functions are not covered with `nonReentrant` modifier from [OZ](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol). It could potentially allow to generate some exploitable scenarios by staking, making some manipulations, and unstaking in the same tx (e.g. in order to repay the flash loan fee). However, the motivation behind not doing this is clear, it consumes a lot of gas for setting/unsetting the mutex flag. If the Asymmetry decides to protect those function, consider reading about the following design which introduces [Re-entrancy Guard 2.0](https://medium.com/spherex-technologies/reentrancy-guard-2-0-cbbc0be41634)
  

</br>

# EIP-4895 Integration Analysis
* ## About the EIP-4895
  * [EIP-4895](https://eips.ethereum.org/EIPS/eip-4895) is about providing an opportunity for validators:
    * to move their CL rewards to EL increasing the capital efficiency. This happens automatically upon an upcoming network fork. 
    * to exit from the beacon chain and fully recieve their staked eth.
  * Therefore, it's pretty important for liquid staking protocols to integrate this feature for a better compliance. 
  
</br>

* ## Withdrawals Architecture Proposed by the Lido
  * We'll consider the withdrawal design proposed by Lido as an example so that the Asymmetry Finance could figure out about building a proper interface to interact with an underlying derivatives. 
  * According to a [feature/shapella-upgrade branch](https://github.com/lidofinance/lido-dao/tree/feature/shapella-upgrade), Lido is intending to process all withdrawals though their withdrawal queue. The higher perspective over the withdrawal feature is defined below:

    ```mermaid
    sequenceDiagram
        participant Lido_Staker
        participant Withdrawal_Queue
        participant Finalize_Role 
        participant LidoELRewards_Vault 
        participant Lido_DAO 
        participant ValidatorExitBus_Oracle
        participant RandomLidoValidator

        Lido_Staker->>Withdrawal_Queue: requests withdrawals by burning(stETH/wstETH)
        Lido_DAO->>ValidatorExitBus_Oracle: exit the validator according to the DAO policy
        ValidatorExitBus_Oracle->>Lido_DAO: sends unstaked eth
        Lido_DAO->>Finalize_Role: sends unstaked eth to process withdrawals
        RandomLidoValidator->>LidoELRewards_Vault: sends rewards after proposing a block on CL
        LidoELRewards_Vault->>Finalize_Role: sends EL and MEV rewards
        Finalize_Role->>Withdrawal_Queue: finalizes the batch of withdrawals
        Lido_Staker->>Withdrawal_Queue: claims withdrawals
        Withdrawal_Queue->>Lido_Staker: sends some fresh eth
    ```
  
  * For Asymmetry Finance it will be quite challenging to process withdrawals, since every LSD has its own difficulties. In a case with Lido, we can see that it takes some time to process a single withdrawal, cuz it should be finalized first. Since Asymmetry Finance is built on top of several LSDs, we can't simply ask for `derivative[i].balance() * _safEthAmount / safEthTotalSupply` and withdraw that amount immediately, therefore we need some aggregated solution here. Some straight forward idea to deal with that could be about leveraging some special reserves allocated only for processing withdrawals, but the capital efficiency of those reserves is not on a high level.

</br>

* ## Additional Caveats
  * Some extra delays between requesting and claiming withdrawals might occur in case if the validators of LSDs perform very poorly. Although it might be an unexpected case, where the client running by validators might have a bug, which might be the case for slashing. Lido has a special mechanism called "bunker mode" to deal with a negative rebase(under a massive slashing). Once the bunker mode is enabled, it takes about 36 days to enter a "turbo mode" back. Asymmetry Finance should consider some unexpected cases like that to mitigate the risks as much as possible.
  
  * Currently the protocol is heavily dependent on a Curve pools in order to perform some withdrawals. However, Asymmetry Finance should encounter the market instabilities. Let's say, AccountingOracle in Lido is about to submit the report with a negative rebase. After simulating this transaction, some mev bots will try to front-run this tx by dumping stETH in order to exchange for eth, because after a negative rebase happens, stETH could inflate a little, since the shares are the same, but the underlying eth decresed(as a result of slashing). After the report is submitted the stakers also will rush to dump their stETH on Curve. This results in having less and less liquidity in Curve pool, since everyone is trying to save their stETH. Now, the developers behind Asymmetry see this wild moment and they decide to set the weight for Lido to 0. After that the rebalance has to be called, but unfortunately, due to the slippage, it's not possible to exchange the stETH held by Asymmetry Finance. You can find some Wild market discussions with a Curve Founder here: 
  https://t.me/ETHSecurity/68819
   
# Asymmetry Improvement Proposals


* ## Low Severity Issues
  * **[[AIPL-01] Add an extra check when depositing to RocketPool derivative]()**
  * **[[AIPL-02] The future growth problems associated with the number underlying of derivatives]()**
  * **[[AIPL-03] The risks behind adjusting the weight for non-existing derivative]()**
  * **[[AIPL-04] It's possible to set the max slippage for non-existing derivative]()**
  * **[[AIPL-05] The underlying wstETH could be trapped, since the stETH is pausable]()**
  * **[[AIPL-06] Pause `stake()`/`unstake()` untill the protocol is ready to process]()**
  * **[[AIPL-07] An assertion over `totalSupply() > 0` seems relevant upon unstaking]()**
  * **[[AIPL-08] Unchecked return value upon `IERC20().approve/transfer/transferFrom`]()**
</br>

* ## Non-Critical Severity Issues

</br>

## **[AIPL-01] Add an extra check when depositing to RocketPool derivative**<a name="AIPL-01"></a>

- In [Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol) there is a check to identify an ability to deposit directly to the pool. Currently this check consist of two parts: 
  - If the `msg.value >= rocketDAOProtocolSettingsDeposit.getMinimumDeposit()`
  - If the `rocketDepositPool.getBalance() + _amount <= rocketDAOProtocolSettingsDeposit.getMaximumDepositPoolSize()`

  - And there is no check for whether deposits are enabled or not. Which is presented here: 
https://etherscan.io/address/0x2cac916b2A963Bf162f076C0a8a4a8200BCFBfb4#code#F7#L103 

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L146-L149

### ***Recommendations***

- Short term: 
  - Consider placing the following check described above.
- Long term: N/A

</br>

## **[AIPL-02] The future growth problems associated with the number of underlying derivatives**<a name="AIPL-02"></a>

-  Adding new derivates leads to an increased gas consumption for both `stake()` and `unstake()` functions. Therefore, it's not reasonable for the each `stake()`/`unstake()` to spend several millions of gas units.

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108-L129
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63-L101

### ***Recommendations***

- Short term: N/A
- Long term: Consider this submission as a note to think about the troubles that could occur in the future. Unfortunately, there is no an easy solution here, since it probably requires a little architecture re-design. 

</br>

## **[AIPL-03] The risks behind adjusting the weight for non-existing derivative**<a name="AIPL-03"></a>

-  In [SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol) there is a function `adjustWeight()` which is used to change the derivative weight. However, there is no an invariant placed to check whether `derivatives[_derivativeIndex]` exists or not, hence there is a risk of setting the weight for non-existing derivative. 

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175

### ***Recommendations***

- Short term: 
  - In order to mitigate that risk, consider placing an appropriate assertion. In order to avoid additional warm-access SLOAD, cache the `derivativeCount` into the memory.
- Long term: N/A

</br>

## **[AIPL-04] It's possible to set the max slippage for non-existing derivative**<a name="AIPL-04"></a>

- In [SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol) there is a function `setMaxSlippage()` for managing the minOut amount upon processing various exchanges in a secondary market. The owner might accidentally set the max slippage for non-existing derivative thinking that everything is okay. The flow could look like that: 

    ```mermaid
    sequenceDiagram
        participant SafEth_Owner
        participant rebalanceToWeights()
        participant RandomDerivative
        participant CurvePool
        participant setMaxSlippage()

        SafEth_Owner->>rebalanceToWeights(): tries to rebalance the weights
        rebalanceToWeights()->>RandomDerivative: sends withdrawal request  
        RandomDerivative->>CurvePool: tries to exchange with a pre-setted slippage
        CurvePool->>RandomDerivative: fails because of the slippage >1%
        RandomDerivative->>rebalanceToWeights(): reports the failure
        SafEth_Owner->>setMaxSlippage(): slightly increases the slippage in rush, but accidentally for non-existing derivative 
        SafEth_Owner->>rebalanceToWeights(): tries again to rebalance the weights
        rebalanceToWeights()->>RandomDerivative: sends withdrawal request  
        RandomDerivative->>CurvePool: tries to exchange with a pre-setted slippage
        CurvePool->>RandomDerivative: fails because of the slippage >1%
        RandomDerivative->>rebalanceToWeights(): reports the failure
        SafEth_Owner->>SafEth_Owner: is upset now, since he losts some eth for failures
    ```
- Of course, it happens in theory, but very low chance that this scenario comes in reality. Anyways, an invariant placed to prevent such risks is absolutely relevant.

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202-L208
### ***Recommendations***
- Short term: Place an appropriate assertion to prevent that from happening.
- Long term: N/A

</br>

## **[AIPL-05] The underlying wstETH could be trapped, since the stETH is pausable**<a name="AIPL-05"></a>

-  Although, it's clear that the dApps should work with wstETH, since the stETH is rebasable(check out the stETH/wstETH integration [guide](https://docs.lido.fi/guides/steth-integration-guide/)), however, in this case, there is a small risk behind trapping all the underlying wstETH if the stETH is paused by an emergency case.

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L59

### ***Recommendations***

- Short term: 
  - Provide a way for stakers to access their wstETH, even if the stETH is paused. The stakers could use wstETH directly to get a collateralized loan on Aave or MakerDAO or simply exchange using AMM pools.
- Long term: N/A

</br>


## **[AIPL-06] Pause `stake()`/`unstake()` untill the protocol is ready to process**<a name="AIPL-06"></a>

- It's pretty popular to pause the key functionality untill everything settles down correctly. It helps to mitigate some possible shares manipulation risks due to the fact that not everything is setted correctly at some point of time. Or in some cases, there might be an oppotunity to front-run the tx which sets something crucial. 
  
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L48-L56

### ***Recommendations***
- Short term: 
  - Consider pausing an opportunity to stake/unstake in `initialize()` untill the protocol is ready to operate.
  
- Long term: N/A 

</br>

## **[AIPL-07] `totalSupply()` should be >0 upon unstaking**<a name="AIPL-07"></a>

- An extra invariant could be placed upon unstaking to make sure that at least one staking was performed before an unstake.
  
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L109-L111

### ***Recommendations***
- Short term: 
  - Consider the following assertion:
    ```Solidity
      require(pauseUnstaking == false, "unstaking is paused");
      uint256 safEthTotalSupply = totalSupply();
      uint256 ethAmountBefore = address(this).balance;
      assert(safEthTotalSupply > 0);
    ```
  
- Long term: N/A 

</br>

## **[AIPL-08] Unchecked return value upon `IERC20().approve/transfer/transferFrom`**<a name="AIPL-08"></a>
- According to the [JPL Institutional Coding Standard for the C Programming Language](https://www.youtube.com/watch?v=Wm3t8Fuiy1E&t=86s), quote, every return value must be used or explicitly discarded. 

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L59
### ***Recommendations***
- Short term: Consider placing corresponding assertion upon `IERC20().approve()`.
- Long term: N/A 

</br>