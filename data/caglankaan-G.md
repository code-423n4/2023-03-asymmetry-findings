# Using Prefix Operators Costs Less Gas Than Postfix Operators in Loops

### Description
In Solidity, there are two ways to increment a variable within a loop: i++ and ++i. The difference between them is that i++ is a post-increment operator, which increments the value of i after the expression has been evaluated, while ++i is a pre-increment operator, which increments the value of i before the expression has been evaluated.

When using these operators in a loop, it is more gas-efficient to use ++i instead of i++. The reason is that the i++ operator requires an additional SSTORE operation to store the updated value of i, which consumes more gas.

### Code Location 

Applies for all of the for loops
```solidity
for (uint i = 0; i < derivativeCount; i++)
```


### Recommendation
It is recommended to use ++i instead of i++ to increment the value of a uint variable within a loop. This is not applicable outside of loops.


# Avoid Unnecessary Initializations Of Uint256 And Bool Variable To 0/false
This code optimization involves avoiding the unnecessary initialization of uint256 and bool variables to 0 or false. In some cases, the code block initializes a variable to 0 or false even though it is already initialized to the same value by default, which leads to unnecessary gas consumption. This code optimization can be applied to several for loops and conditional statements in different Solidity smart contracts.



### Code Example

Applies for all of the for loops
```solidity
for (uint i = 0; i < derivativeCount; i++) {
for (uint256 i = 0; i < derivativeCount; i++) {
```

### Recommendation
To optimize the code and reduce gas consumption, it is recommended to avoid initializing uint and bool variables to 0 or false when they are already initialized to the same value by default. Instead, simply declare the variable without an initial value, as in for (uint256 i; i < length; ++i) or bool isDone;. This will make the code more efficient and optimize the gas usage. The developers should review their Solidity smart contracts to identify and apply this code optimization.


# Avoiding Unnecessary State Changes and Events

## Description

One way to optimize the gas usage of a smart contract is to minimize the number of state changes and events emitted. Each state change and event emit incurs a certain amount of gas cost, so reducing the number of unnecessary state changes and events can significantly reduce the overall gas cost of executing the contract.

## Code Example

```solidity
    /**
        @notice - Sets the minimum amount a user is allowed to stake
        @param _minAmount - amount to set as minimum stake value
    */
    function setMinAmount(uint256 _minAmount) external onlyOwner {
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }

    /**
        @notice - Owner only function that sets the maximum amount a user is allowed to stake
        @param _maxAmount - amount to set as maximum stake value
    */
    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
    }

    /**
        @notice - Owner only function that Enables/Disables the stake function
        @param _pause - true disables staking / false enables staking
    */
    function setPauseStaking(bool _pause) external onlyOwner {
        pauseStaking = _pause;
        emit StakingPaused(pauseStaking);
    }

    /**
        @notice - Owner only function that enables/disables the unstake function
        @param _pause - true disables unstaking / false enables unstaking
    */
    function setPauseUnstaking(bool _pause) external onlyOwner {
        pauseUnstaking = _pause;
        emit UnstakingPaused(pauseUnstaking);
    }
```

## Recommendation
To minimize gas usage and optimize the performance of a smart contract, it is recommended to carefully design the contract to minimize state changes and events. Use local variables instead of global variables when possible, and avoid redundant or unnecessary code. Additionally, thoroughly test and optimize the contract to ensure that it is efficient and secure.

Example:

```solidity
function setMinAmount(uint256 _minAmount) external onlyOwner {
    minAmount = _minAmount;
    emit ChangeMinAmount(_minAmount);
}
```


# Missing Re-entrancy Protection
Re-entrancy is a vulnerability that occurs when a contract makes an external call to an untrusted contract, which then calls back into the original contract before the first call has completed. This can lead to unexpected behavior, including re-entrancy attacks where the attacker repeatedly re-enters the original contract, exploiting it to extract more funds or perform unauthorized actions.

One common way to prevent re-entrancy attacks is to use a mutex lock. A mutex lock is a simple mechanism that allows a contract to ensure that only one execution path can access critical code at any given time. By using a mutex lock, a contract can prevent any external call from interrupting its execution and ensure that all state changes are committed before any new external call can be made.

## Code Example

```solidity
    /**
        @notice - Unstake your safETH into ETH
        @dev - unstakes a percentage of safEth based on its total value
        @param _safEthAmount - amount of safETH to unstake into ETH
    */
    function unstake(uint256 _safEthAmount) external {
        require(pauseUnstaking == false, "unstaking is paused");
        uint256 safEthTotalSupply = totalSupply();
        uint256 ethAmountBefore = address(this).balance;

        for (uint256 i = 0; i < derivativeCount; i++) {
            // withdraw a percentage of each asset based on the amount of safETH
            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
            if (derivativeAmount == 0) continue; // if derivative empty ignore
            derivatives[i].withdraw(derivativeAmount);
        }
        _burn(msg.sender, _safEthAmount);
        uint256 ethAmountAfter = address(this).balance;
        uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;
        // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
            ""
        );
        require(sent, "Failed to send Ether");
        emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
    }
```

## Recommendation
To prevent re-entrancy attacks, it is recommended to use a mutex lock to ensure that only one execution path can access critical code at any given time. One common pattern is to use the `ReentrancyGuard` contract from OpenZeppelin, which provides a `nonReentrant` modifier that can be applied to any function that requires protection.



