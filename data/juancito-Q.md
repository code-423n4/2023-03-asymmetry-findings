# QA Report

## Summary

### Low Risk Issues

- [L-1] Users Ether can be locked in contract if sent by mistake
- [L-2] Two-step ownership transfer
- [L-3] Add safety checks to admin functions

### Non-Critical Issues

- [NC-1] Add support for path mappings
- [NC-2] Floating Pragma version
- [NC-3] Replace uint with uint256
- [NC-4] Use custom errors instead of messages
- [NC-5] Add curly braces for control structures
- [NC-6] Factors can be canceled
- [NC-7] Missing NatSpec

## Low Risk Issues

### [L-1] Users Ether can be locked in contract if sent by mistake

#### Proof of Concept

The `SafEth` contract and derivatives contracts have a `receive` function that allows users to easily send Ether by mistake when trying to interact with them. This is not uncommon and can be prevented. Upgrading the contracts to do so would be difficult as there is no on-chain record of those transfers.

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L246

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L244

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L126

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L97

#### Recommended Mitigation Steps

Prevent the contracts from receiving Ether from any address except the ones that they are expected to interact with.

In the case of the `SafEth`, require it to only receive Ether from the derivatives contract addresses.

In the case of the derivatives, require them to only receive Ether from the `SafErh` contract address, and from the pools and contracts they interact with.

### [L-2] Two-step ownership transfer

The `owner` role in the `SafEth` contract and the derivatives contracts is critical to perform operations on the platform. In order to prevent transfering it to a wrong address by mistake, it is recommended to perform the transfer in two steps.

#### Proof of Concept

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L11

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L13

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L6

- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L5

#### Recommended Mitigation Steps

Use [Ownable2StepUpgradeable](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol) instead of `OwnableUpgradeable` that achieves the same original functionality with the addition of a method to confirm the the ownership.

### [L-3] Add safety checks to admin functions

Safety checks can prevent admin errors that could greatly harm the project, at the expense of a cheap check.

Prevent adding a derivative contract with `address(0)`, which would break `stake` and `unstake` functions.

```diff
// File: contracts/SafEth/SafEth.sol

    function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
+       require(_contractAddress != 0, "Cannot be address(0)");
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;

        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    }
```

Prevent setting an excesive slippage value that can affect all trades, by defining `MAX_SLIPPAGE`:

```diff
// File: contracts/SafEth/SafEth.sol

    function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
    ) external onlyOwner {
+       require(_slippage <= MAX_SLIPPAGE, "Has to be lower than MAX_SLIPPAGE");
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
    }
```

Check that the min amount value is <= than the max amount value, to prevent breaking `stake` and `unstake` functions:

```diff
// File: contracts/SafEth/SafEth.sol

    function setMinAmount(uint256 _minAmount) external onlyOwner {
+       require(_minAmount <= maxAmount, "minAmount has to be less than maxAmount");
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }

    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
+       require(_maxAmount >= minAmount, "maxAmount has to be greater than minAmount");
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
    }
```

## Non-Critical Issues

### [NC-1] Add support for path mappings

Consider adding support for path mappings in order to have a clear understanding of located dependencies and in favor of clean code.

This will allow to have imports like this:

```diff
- import "../../interfaces/IDerivative.sol";
+ import "contracts/interfaces/IDerivative.sol";
```

Intructions on how to do so are in [Hardhat documentation](https://hardhat.org/hardhat-runner/docs/guides/typescript#support-for-path-mappings).

### [NC-2] Floating Pragma version

Contracts in scope have a floating Pragma version `pragma solidity ^0.8.13;`.

With a floating pragma, contracts may accidentally be deployed using an outdated or problematic compiler version which can cause bugs.

Consider fixing it to a specific value like for example `pragma solidity 0.8.13;`.

### [NC-3] Replace uint with uint256

Contracts in scope use both `uint` and `uint256` indistinctively.

In order to improve consistency of the codebase, consider sticking to one of them for convention.

`uint` instances:

```solidity
// File: contracts/SafEth/SafEth.sol

26:     event Staked(address indexed recipient, uint ethIn, uint safEthOut);
27:     event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);
28:     event WeightChange(uint indexed index, uint weight);
29:     event DerivativeAdded(
30:         address indexed contractAddress,
31:         uint weight,
32:         uint index
33:     );

71:     for (uint i = 0; i < derivativeCount; i++)

84:     for (uint i = 0; i < derivativeCount; i++) {

92:     uint derivativeReceivedEthValue = (derivative.ethPerDerivative(

140:    for (uint i = 0; i < derivativeCount; i++) {

147:    for (uint i = 0; i < derivativeCount; i++) {

203:    uint _derivativeIndex,
204:    uint _slippage
```

```solidity
// File: contracts/SafEth/SafEth.sol

171:    uint rethPerEth = (10 ** 36) / poolPrice();

241:    return (sqrtPriceX96 * (uint(sqrtPriceX96)) * (1e18)) >> (96 * 2);
```

### [NC-4] Use custom errors instead of messages

From [Solidity documentation](https://docs.soliditylang.org/en/v0.8.19/structure-of-a-contract.html#errors):

"Errors allow you to define descriptive names and data for failure situations. Errors can be used in revert statements. In comparison to string descriptions, errors are much cheaper and allow you to encode additional data. You can use NatSpec to describe the error to the user."

Consider using custom errors for:

```solidity
// File: contracts/SafEth/SafEth.sol

54:         require(pauseStaking == false, "staking is paused");
55:         require(msg.value >= minAmount, "amount too low");
56:         require(msg.value <= maxAmount, "amount too high");

99:         require(pauseUnstaking == false, "unstaking is paused");

117:        require(sent, "Failed to send Ether");
```

```solidity
// File: contracts/SafEth/derivatives/Reth.sol

113:        require(sent, "Failed to send Ether");

200:        require(rethBalance2 > rethBalance1, "No rETH was minted");
```

```solidity
// File: contracts/SafEth/derivatives/SfrxEth.sol

87:         require(sent, "Failed to send Ether");
```

```solidity
// File: contracts/SafEth/derivatives/WstEth.sol

66:         require(sent, "Failed to send Ether");

77:         require(sent, "Failed to send Ether");
```

### [NC-5] Add curly braces for control structures

It improves readability and code clarity.

From [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.19/style-guide.html#control-structures):

```
The braces denoting the body of a contract, library, functions and structs should:

- open on the same line as the declaration
- close on their own line at the same indentation level as the beginning of the declaration.
- The opening brace should be preceded by a single space.
```

Consider adding curly braces to the following control structures:

```solidity
// File: contracts/SafEth/SafEth.sol

71:        for (uint i = 0; i < derivativeCount; i++)
72:            underlyingValue +=
73:                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74:                    derivatives[i].balance()) /
75:                10 ** 18;

79:        if (totalSupply == 0)
80:            preDepositPrice = 10 ** 18; // initializes with a price of 1

87:        if (weight == 0) continue;

117:       if (derivativeAmount == 0) continue; // if derivative empty ignore

141:       if (derivatives[i].balance() > 0)
142:           derivatives[i].withdraw(derivatives[i].balance());

148:       if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
```

```solidity
// File: contracts/SafEth/derivatives/Reth.sol

170:        if (!poolCanDeposit(msg.value)) {
171:            uint rethPerEth = (10 ** 36) / poolPrice();

212:        if (poolCanDeposit(_amount))
213:            return
214:                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
215:        else return (poolPrice() * 10 ** 18) / (10 ** 18);
```

### [NC-6] Factors can be canceled

Remove unnecessary calculations of factors than can be canceled in `Reth.ethPerDerivative()`:

```diff
- else return (poolPrice() * 10 ** 18) / (10 ** 18);
+ else return poolPrice();
```

[Link to code](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215)

### [NC-7] Missing NatSpec

Missing `amount` NatSpec in `Reth.withdraw()`:

```solidity
/**
    @notice - Convert derivative into ETH
    */
function withdraw(uint256 amount) external onlyOwner {
```

Missing `_slippage` NatSpec in `SfrxEth.setMaxSlippage()`:

```solidity
    /**
        @notice - Owner only function to set max slippage for derivative
    */
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

Missing `_slippage` NatSpec in `WstEth.setMaxSlippage()`:

```solidity
    /**
        @notice - Owner only function to set max slippage for derivative
    */
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
```

Missing `_amount` NatSpec in `WstEth.withdraw()`:

```solidity
    /**
        @notice - Owner only function to Convert derivative into ETH
        @dev - Owner is set to SafEth contract
     */
    function withdraw(uint256 _amount) external onlyOwner {
```