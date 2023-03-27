| Total Low issues |
|------------------|

| Risk   | Issues Details                                                                                              | Number        |
|--------|-------------------------------------------------------------------------------------------------------------|---------------|
| [L-01] | No Storage Gap for Upgradeable contracts                                                                    | 4             |
| [L-02] | Loss of precision due to rounding                                                                           | 11            |
| [L-03] | Lack of `nonReentrant` modifier                                                                             | 1             |
| [L-04] | Missing Event for initialize                                                                                | 4             |
| [L-05] | `owner` can renounce while system is paused                                                                 | 1             |
| [L-06] | Unused `receive()` Function Will Lock Ether In Contract                                                     | 4             |
| [L-07] | Use a more recent version of OpenZeppelin dependencies                                                      | 1             |
| [L-08] | Value is not validated to be different than the existing one                                                | 5             |
| [L-09] | Add a timelock to critical functions                                                                        | 8             |
| [L-10] | Lock pragmas to specific compiler version                                                                   | 4             |
| [L-11] | Use `uint256` instead `uint`                                                                                | 16            |
| [L-12] | Inconsistent check between Reth.Deposit() and WstEth.deposit(), SfrxEth.deposit()                           | 2             |
| [L-13] | Critical changes should use-two step procedure                                                              | 4             |

| Total Non-Critical issues |
|---------------------------|

| Risk    | Issues Details                                                                                             | Number        |
|---------|------------------------------------------------------------------------------------------------------------|---------------|
| [NC-01] | Include return parameters in NatSpec comments                                                              | All Contracts |
| [NC-02] | Non-usage of specific imports                                                                              | All Contracts |
| [NC-03] | Lack of event emit                                                                                         | 3             |
| [NC-04] | Function writing does not comply with the `Solidity Style Guide`                                           | All Contracts |
| [NC-05] | Solidity compiler optimizations can be problematic                                                         | 1             |
| [NC-06] | Use `bytes.concat()` and `string.concat()`                                                                 | 6             |
| [NC-07] | The protocol should include NatSpec                                                                        | All Contracts |
| [NC-08] | Constants in comparisons should appear on the left side                                                    | 5             |
| [NC-09] | Use a more recent version of solidity                                                                      | 4             |
| [NC-10] | Contracts should have full test coverage                                                                   | All Contracts |
| [NC-11] | Need Fuzzing test                                                                                          | All Contracts |
| [NC-12] | Generate perfect code headers every time                                                                   | All Contracts |
| [NC-13] | For functions, follow Solidity standard naming conventions                                                 | All Contracts |
| [NC-14] | Events that mark critical parameter changes should contain both the old and the new value                  | 5             |
| [NC-15] | There is no need to cast a variable that is already an address, such as `address(x)`                       | 4             |
| [NC-16] | Use scientific notation rather than exponentiation                                                         | 26            |

## [L-01] No Storage Gap for Upgradeable contracts

#### Description

For upgradeable contracts, inheriting contracts may introduce new variables. In order to be able to add new variables to the upgradeable contract without causing storage collisions, a storage gap should be added to the upgradeable contract.

#### Lines of code 

- [WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol)

- [SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol)

- [Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol)

- [SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)

#### Recommended Mitigation Steps

Consider adding a storage gap at the end of the upgradeable contract:
```solidity
  /**
   * @dev This empty reserved space is put in place to allow future versions to add new
   * variables without shifting down storage in the inheritance chain.
   * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
   */
  uint256[50] private __gap;
```

## [L-02] Loss of precision due to rounding

#### Description

Loss of precision due to the nature of arithmetics and rounding errors.

#### Lines of code 

```solidity
        uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
```

- [WstEth.sol#L60](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L60)

```solidity
        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
            (10 ** 18 - maxSlippage)) / 10 ** 18;
```

- [SfrxEth.sol#L74-L75](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75)

```solidity
        return ((10 ** 18 * frxAmount) /
            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
```

- [SfrxEth.sol#L115-L116](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L115-L116)

```solidity
        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
```

- [SafEth.sol#L81](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L81)

```solidity
            uint256 ethAmount = (msg.value * weight) / totalWeight;
```

- [SafEth.sol#L88](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L88)

```solidity
        uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```

- [SafEth.sol#L98](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L98)

```solidity
            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
```

- [SafEth.sol#L115-L116](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L115-L116)

```solidity
            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                totalWeight;
```

- [SafEth.sol#L149-L150](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L149-L150)

```solidity
            uint rethPerEth = (10 ** 36) / poolPrice();
```

- [Reth.sol#L171](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L171)

```solidity
            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
                ((10 ** 18 - maxSlippage))) / 10 ** 18);
```

- [Reth.sol#L173-L174](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173-L174)

```solidity
        else return (poolPrice() * 10 ** 18) / (10 ** 18);
``` 

- [Reth.sol#L215](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215)

## [L-03] Lack of `nonReentrant` modifier

#### Description

It is a best practice to use the `nonReentrant` modifier when you make external calls to untrustable entities because even if an auditor did not think of a way to exploit it, an attacker just might.

#### Lines of code 

```solidity
        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
            ""
        );
```

- [SafEth.sol#L124-L126](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L124-L126)

#### Recommended Mitigation Steps

Add reentrancy guard to the functions mentioned above.

## [L-04] Missing Event for initialize

#### Description

Events help non-contract tools to track changes, and events prevent users from being surprised by changes Issuing event-emit during initialization is a detail that many projects skip.

#### Lines of code 

```solidity
    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }
```

- [WstEth.sol#L33-L36](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L33-L36)

```solidity
    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }
```

- [SfrxEth.sol#L36-L39](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L36-L39)

```solidity
    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }
```

- [Reth.sol#L42-L45](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L42-L45)

```solidity
    function initialize(
        string memory _tokenName,
        string memory _tokenSymbol
    ) external initializer {
        ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
        _transferOwnership(msg.sender);
        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
    }
```

- [SafEth.sol#L48-L56](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L48-L56)

#### Recommended Mitigation Steps

Add Event-Emit

## [L-05] `owner` can renounce while system is paused

#### Description

The contract `owner` is not prevented from renouncing the role/ownership while the contract is paused, which would cause any user assets stored in the protocol, to be locked indefinitely.

#### Lines of code 

- [SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)

#### Recommended Mitigation Steps

Prevent the owner from renouncing the role/ownership while the staking or the unstaking is paused.

## [L-06] Unused `receive()` Function Will Lock Ether In Contract

#### Description

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert.

#### Lines of code

```solidity
    receive() external payable {}
```

- [SafEth.sol#L246](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L246)

```solidity
    receive() external payable {}
```

- [Reth.sol#L244](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L244)

```solidity
    receive() external payable {}
```

- [SfrxEth.sol#L126](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L126)

```solidity
    receive() external payable {}
```

- [WstEth.sol#L97](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L97)

#### Recommended Mitigation Steps

The function should call another function, otherwise it should revert.

## [L-07] Use a more recent version of OpenZeppelin dependencies

#### Description

For security, it is best practice to use the latest OpenZeppelin version.

#### Lines of code 

```solidity
    "@openzeppelin/contracts": "^4.8.0",
```

- [package.json#L82](https://github.com/code-423n4/2023-03-asymmetry/blob/main/package.json#L82)

#### Recommended Mitigation Steps

Old version of OpenZeppelin is used `(4.8.0)`, newer version can be used [`(4.8.2)`](https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.8.2).

## [L-08] Value is not validated to be different than the existing one

#### Description

Value is not validated to be different than the existing one. Queueing the same value will cause multiple abnormal events to be emitted, will ultimately result in a no-op situation.

#### Lines of code 

```solidity
    function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
    ) external onlyOwner {
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
    }
```

- [SafEth.sol#L202-L208](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202-L208)

```solidity
    function setMinAmount(uint256 _minAmount) external onlyOwner {
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }
```

- [SafEth.sol#L214-L217](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214-L217)

```solidity
    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
    }
```

- [SafEth.sol#L223-L226](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223-L226)

```solidity
    function setPauseStaking(bool _pause) external onlyOwner {
        pauseStaking = _pause;
        emit StakingPaused(pauseStaking);
    }
```

- [SafEth.sol#L232-L235](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232-L235)

```solidity
    function setPauseUnstaking(bool _pause) external onlyOwner {
        pauseUnstaking = _pause;
        emit UnstakingPaused(pauseUnstaking);
    }
```

- [SafEth.sol#L241-L244](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241-L244)

#### Recommended Mitigation Steps

Add a `require()` statement to check that the new value is different than the current one.

## [L-09] Add a timelock to critical functions

#### Description

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate.

#### Lines of code 

```solidity
    function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
    ) external onlyOwner {
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
    }
```

- [SafEth.sol#L202-L208](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202-L208)

```solidity
    function setMinAmount(uint256 _minAmount) external onlyOwner {
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }
```

- [SafEth.sol#L214-L217](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214-L217)

```solidity
    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
    }
```

- [SafEth.sol#L223-L226](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223-L226)

```solidity
    function setPauseStaking(bool _pause) external onlyOwner {
        pauseStaking = _pause;
        emit StakingPaused(pauseStaking);
    }
```

- [SafEth.sol#L232-L235](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232-L235)

```solidity
    function setPauseUnstaking(bool _pause) external onlyOwner {
        pauseUnstaking = _pause;
        emit UnstakingPaused(pauseUnstaking);
    }
```

- [SafEth.sol#L241-L244](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241-L244)

```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

- [Reth.sol#L58-L60](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58-L60)

```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

- [SfrxEth.sol#L51-L53](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53)

```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

- [WstEth.sol#L48-L50](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48-L50)

#### Recommended Mitigation Steps

Consider adding a timelock to the critical changes.

## [L-10] Lock pragmas to specific compiler version

#### Description

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally. 

> Ref: https://swcregistry.io/docs/SWC-103

#### Lines of code 

```solidity
pragma solidity ^0.8.13;
```

- [WstEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L2)

```solidity
pragma solidity ^0.8.13;
```

- [SfrxEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L2)

```solidity
pragma solidity ^0.8.13;
```

- [Reth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2)

```solidity
pragma solidity ^0.8.13;
```

- [SafEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2)

#### Recommended Mitigation Steps

[Ethereum Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/): Lock pragmas to specific compiler version. 

## [L-11] Use `uint256` instead `uint`

#### Description

`16 results in 2 files`.

Some developers prefer to use `uint256` because it is consistent with other `uint` data types, which also specify their size, and also because making the size of the data explicit reminds the developer and the reader how much data they've got to play with, which may help prevent or detect bugs.

For example if doing `bytes4(keccak('transfer(address, uint)’))`, you'll get a different method sig ID than `bytes4(keccak('transfer(address, uint256)’))` and smart contracts will only understand the latter when comparing method sig IDs.

#### Lines of code 

```solidity
            uint rethPerEth = (10 ** 36) / poolPrice();
```

- [Reth.sol#L171](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L171)

```solidity
    event Staked(address indexed recipient, uint ethIn, uint safEthOut);
```

- [SafEth.sol#L26](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L26)

```solidity
    event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);
```

- [SafEth.sol#L27](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L27)

```solidity
    event WeightChange(uint indexed index, uint weight);
```

- [SafEth.sol#L28](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L28)

```solidity
        uint weight,
```

- [SafEth.sol#L31](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L31)

```solidity
        uint index
```

- [SafEth.sol#L32](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L32)

```solidity
        for (uint i = 0; i < derivativeCount; i++)
```

- [SafEth.sol#L71](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71)

```solidity
        for (uint i = 0; i < derivativeCount; i++) {
```

- [SafEth.sol#L84](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84)

```solidity
            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
```

- [SafEth.sol#L92](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L92)

```solidity
        for (uint i = 0; i < derivativeCount; i++) {
```

- [SafEth.sol#L140](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140)

```solidity
        for (uint i = 0; i < derivativeCount; i++) {
```

- [SafEth.sol#L147](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147)

```solidity
        uint _derivativeIndex,
```

- [SafEth.sol#L203](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L203)

```solidity
        uint _slippage
```

- [SafEth.sol#L204](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L204)

#### Recommended Mitigation Steps

Use `uint256` instead `uint`.

## [L-12] Inconsistent check between Reth.Deposit() and WstEth.deposit(), SfrxEth.deposit()

#### Description

The following check in [`Reth.Deposit()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L200) function ensures that some Reth were minted to the caller:

```solidity
            require(rethBalance2 > rethBalance1, "No rETH was minted");
```

However, there is no such checks in [`WstEth.deposit()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L73), [`SfrxEth.deposit()`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L94) functions which may lead to unexpected behavior in the future.

#### Lines of code 

- [Reth.sol#L156](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L156)

- [WstEth.sol#L73](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L73)

- [SfrxEth.sol#L94](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L94)

#### Recommended Mitigation Steps

Add the same check to the other functions to make them more robust.

## [L-13] Critical changes should use-two step procedure

#### Description

The following contracts ([`WstEth.sol`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol), [`SfrxEth.sol`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol), [`SafEth.sol`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol), [`Reth.sol`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol)) inherit [`OwnableUpgradeable.sol`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/OwnableUpgradeable.sol) which have a function that allows the owner to transfer ownership to a different address. If the owner accidentally uses an invalid address for which they do not have the private key, then the system will gets locked.

#### Lines of code 

```solidity
    function _transferOwnership(address newOwner) internal virtual {
        address oldOwner = _owner;
        _owner = newOwner;
        emit OwnershipTransferred(oldOwner, newOwner);
    }
```

- [OwnableUpgradeable.sol#L83-L87](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/OwnableUpgradeable.sol#L83-L87)

#### Recommended Mitigation Steps

Consider adding two step procedure on the critical functions where the first is announcing a pending new owner and the new address should then claim its ownership or inherit [`Ownable2StepUpgradeable.sol`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol) instead.

A similar issue was reported in a previous contest and was assigned a severity of medium: code-423n4/2021-06-realitycards-findings#105

## [NC-01] Include return parameters in NatSpec comments

#### Description

If Return parameters are declared, you must prefix them with `/// @return`.
Some code analysis programs do analysis by reading [NatSpec](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html) details, if they can't see the `@return` tag, they do incomplete analysis.

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth)

#### Recommended Mitigation Steps

Include `@return` parameters in NatSpec comments

## [NC-02] Non-usage of specific imports

#### Description

The current form of relative path import is not recommended for use because it can unpredictably pollute the namespace. Instead, the Solidity docs recommend specifying imported symbols explicitly. 

- https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth)

#### Recommended Mitigation Steps

Use specific imports syntax per solidity docs recommendation.

## [NC-03] Lack of event emit

#### Description

The below methods do not emit an event when the state changes, something that it's very important for dApps and users.

#### Lines of code 

```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

- [Reth.sol#L58-L60](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58-L60)

```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

- [SfrxEth.sol#L51-L53](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53)

```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

- [WstEth.sol#L48-L50](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48-L50)

#### Recommended Mitigation Steps

Emit event.

## [NC-04] Function writing does not comply with the `Solidity Style Guide`

#### Description

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

Functions should be grouped according to their visibility and ordered:

- `constructor()`
- `receive()`  
- `fallback()`  
- `external / public / internal / private`
- `view / pure`

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth)

#### Recommended Mitigation Steps

Follow Solidity Style Guide.

## [NC-05] Solidity compiler optimizations can be problematic

#### Description

Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

Exploit Scenario A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

#### Lines of code 

```javaScript
const config: HardhatUserConfig = {
  solidity: {
    version: "0.8.13",
    settings: {
      optimizer: {
        enabled: true,
        runs: 100000,
      },
    },
  },
```

- [hardhat.config.ts#L23-L32](https://github.com/code-423n4/2023-03-asymmetry/blob/main/hardhat.config.ts#L23-L32)

#### Recommended Mitigation Steps

Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug. Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## [NC-06] Use `bytes.concat()` and `string.concat()`

#### Description

Solidity version 0.8.4 introduces: 

- `bytes.concat()` vs `abi.encodePacked(<bytes>,<bytes>)`
- `string.concat()` vs `abi.encodePacked(<string>,<string>)`

https://docs.soliditylang.org/en/v0.8.17/types.html?highlight=bytes.concat#the-functions-bytes-concat-and-string-concat

#### Lines of code 

```solidity
                    abi.encodePacked("contract.address", "rocketTokenRETH")
```

- [Reth.sol#L70](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L70)

```solidity
                    abi.encodePacked("contract.address", "rocketDepositPool")
```

- [Reth.sol#L125](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L125)

```solidity
                    abi.encodePacked(
```

- [Reth.sol#L136](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L136)

```solidity
                    abi.encodePacked("contract.address", "rocketDepositPool")
```

- [Reth.sol#L162](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L162)

```solidity
                        abi.encodePacked("contract.address", "rocketTokenRETH")
```

- [Reth.sol#L191](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L191)

```solidity
                    abi.encodePacked("contract.address", "rocketTokenRETH")
```

- [Reth.sol#L233](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L233)

#### Recommended Mitigation Steps

Use `bytes.concat()` and `string.concat()`

## [NC-07] The protocol should include NatSpec

#### Description

It is recommended that Solidity contracts are fully annotated using NatSpec, it is clearly stated in the Solidity official documentation.

- In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. 

- Some code analysis programs do analysis by reading NatSpec details, if they can't see the tags `(@param, @dev, @return)`, they do incomplete analysis.

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth)

#### Recommended Mitigation Steps

Include [`NatSpec`](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html) comments in the codebase.

## [NC-08] Constants in comparisons should appear on the left side

#### Description

Constants in comparisons should appear on the left side, doing so will prevent typo [bug](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html).

```solidity
            if (weight == 0) continue;
```

#### Lines of code 

```solidity
        if (totalSupply == 0)
```

- [SafEth.sol#L79](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L79)

```solidity
            if (weight == 0) continue;
```

- [SafEth.sol#L87](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L87)

```solidity
            if (derivativeAmount == 0) continue; // if derivative empty ignore
```

- [SafEth.sol#L117](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L117)

```solidity
            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
```

- [SafEth.sol#L148](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L148)

#### Recommended Mitigation Steps

Constants should appear on the left side:

```solidity
            if (0 == weight) continue;
```

## [NC-09] Use a more recent version of solidity

#### Description

For security, it is best practice to use the [latest Solidity version](https://github.com/ethereum/solidity/blob/develop/Changelog.md).

#### Lines of code 

```solidity
pragma solidity ^0.8.13;
```

- [WstEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L2)

```solidity
pragma solidity ^0.8.13;
```

- [SfrxEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L2)

```solidity
pragma solidity ^0.8.13;
```

- [Reth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2)

```solidity
pragma solidity ^0.8.13;
```

- [SafEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2)

#### Recommended Mitigation Steps

Old version of Solidity is used `(^0.8.13)`, newer version can be used `(0.8.19)`.

## [NC-10] Contracts should have full test coverage

#### Description

While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

#### Lines of code 

```solidity
- What is the overall line coverage percentage provided by your tests?:  92
```

- [test](https://github.com/code-423n4/2023-03-asymmetry/tree/main/test)

#### Recommended Mitigation Steps

Line coverage percentage should be 100%.

## [NC-11] Need Fuzzing test

#### Description

As Alberto Cuesta Canada said: Fuzzing is not easy, the tools are rough, and the math is hard, but it is worth it. Fuzzing gives me a level of confidence in my smart contracts that I didn’t have before. Relying just on unit testing anymore and poking around in a testnet seems reckless now.

Ref: https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05

#### Lines of code 

- [test](https://github.com/code-423n4/2023-03-asymmetry/tree/main/test)

#### Recommended Mitigation Steps

Use should fuzzing test like Echidna.

## [NC-12] Generate perfect code headers every time

#### Description

I recommend using header for Solidity code layout and readability

> Ref: https://github.com/transmissions11/headers

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth)

#### Recommended Mitigation Steps

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## [NC-13] For functions, follow Solidity standard naming conventions

#### Description

The protocol don't follow solidity standard naming convention.

> Ref: https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth)

#### Recommended Mitigation Steps

Follow solidity standard [naming convention](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions).

## [NC-14] Events that mark critical parameter changes should contain both the old and the new value

#### Description

Events that mark critical parameter changes should contain both the old and the new value.

#### Lines of code 

```solidity
    function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
    ) external onlyOwner {
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
    }
```

- [SafEth.sol#L202-L208](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202-L208)

```solidity
    function setMinAmount(uint256 _minAmount) external onlyOwner {
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }
```

- [SafEth.sol#L214-L217](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214-L217)

```solidity
    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
    }
```

- [SafEth.sol#L223-L226](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223-L226)

```solidity
    function setPauseStaking(bool _pause) external onlyOwner {
        pauseStaking = _pause;
        emit StakingPaused(pauseStaking);
    }
```

- [SafEth.sol#L232-L235](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232-L235)

```solidity
    function setPauseUnstaking(bool _pause) external onlyOwner {
        pauseUnstaking = _pause;
        emit UnstakingPaused(pauseUnstaking);
    }
```

- [SafEth.sol#L241-L244](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241-L244)

```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

- [Reth.sol#L58-L60](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58-L60)

```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

- [SfrxEth.sol#L51-L53](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53)

```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

- [WstEth.sol#L48-L50](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48-L50)

#### Recommended Mitigation Steps

Add the old value to the event.

## [NC-15] There is no need to cast a variable that is already an address, such as `address(x)`

#### Description

There is no need to cast a variable that is already an `address`, such as `address(x)`, `x` is also `address`.

```solidity
        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
            ""
        );
```

#### Lines of code

```solidity
        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
```

- [SafEth.sol#L124](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L124)

```solidity
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
```

- [WstEth.sol#L63](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L63)

```solidity
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
```

- [SfrxEth.sol#L84](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L84)

```solidity
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
```

- [Reth.sol#L110](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L110)

#### Recommended Mitigation Steps     

```solidity
        (bool sent, ) = msg.sender.call{value: ethAmountToWithdraw}(
            ""
        );
```

## [NC-16] Use scientific notation rather than exponentiation

#### Description

While the compiler knows to optimize away the exponentiation, it's still better coding practice to use idioms that do not require compiler optimization, if they exist.

#### Lines of code 

```solidity
        maxSlippage = (1 * 10 ** 16); // 1%
```

- [WstEth.sol#L35](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L35)

```solidity
        uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
```

- [WstEth.sol#L60](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L60)

```solidity
        return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
```

- [WstEth.sol#L87](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L87)

```solidity
        maxSlippage = (1 * 10 ** 16); // 1%
```

- [SfrxEth.sol#L38](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L38)

```solidity
        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
            (10 ** 18 - maxSlippage)) / 10 ** 18;
```

- [SfrxEth.sol#L74-L75](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75)

```solidity
        uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
            10 ** 18
```

- [SfrxEth.sol#L112-L113](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L112-L113)

```solidity
        return ((10 ** 18 * frxAmount) /
```

- [SfrxEth.sol#L115](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L115)

```solidity
        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
```

- [SafEth.sol#L54](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L54)

```solidity
        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
```

- [SafEth.sol#L55](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L55)

```solidity
                    derivatives[i].balance()) /
                10 ** 18;
```

- [SafEth.sol#L74-L75](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L74-L75)

```solidity
            preDepositPrice = 10 ** 18; // initializes with a price of 1
```

- [SafEth.sol#L80](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L80)

```solidity
        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
```

- [SafEth.sol#L81](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L81)

```solidity
            ) * depositAmount) / 10 ** 18;
```

- [SafEth.sol#L94](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L94)

```solidity
        uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```

- [SafEth.sol#L98](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L98)

```solidity
        maxSlippage = (1 * 10 ** 16); // 1%
```

- [Reth.sol#L44](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L44)

```solidity
            uint rethPerEth = (10 ** 36) / poolPrice();
```

- [Reth.sol#L171](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L171)

```solidity
            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
                ((10 ** 18 - maxSlippage))) / 10 ** 18);
```

- [Reth.sol#L173-L174](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173-L174)

```solidity
                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
```

- [Reth.sol#L214](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L214)

```solidity
        else return (poolPrice() * 10 ** 18) / (10 ** 18);
```

- [Reth.sol#L215](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215)

#### Recommended Mitigation Steps

Use scientific notation `(e.g. 1e18)` rather than exponentiation `(e.g. 10 ** 18)`.
