### Issues 
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| O | Ordinary | Often found issues |

| Total Found Issues | 21 |
|:--:|:--:|

### Low Risk
| Count | Explanation | Instances |
|:--:|:-------|:--:| 
| [L-01] | NO USE OF UPGRADEABLE IERC20 | 3 |
| [L-02] | CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE | 4 |
| [L-03] | MISSING EVENTS FOR ONLY FUNCTIONS THAT CHANGE CRITICAL PARAMETERS | 6 |
| [L-04] | THE FUNCTIONS OF THE PROTOCOL ARE SERIOUSLY COMPROMISED | 4 |
| [L-05] | IN THE EMITS, INCLUDE THE OLD AND NEW VALUES OF THE UPDATED PARAMETERS TO TRACK THE CHANGES MADE | 3 |

| Total Low Risk Issues | 5 | Total Instances | 19 |
|:--:|:--:|:--:|--:|

### Non-Critical
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE | All contracts |
| [N-02] | USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS | 3 |
| [N-03] | NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES | all contracts |
| [N-04] | USE A MORE RECENT VERSION OF SOLIDITY | All contracts |
| [N-05] | CREATE YOUR OWN IMPORT NAMES INSTEAD OF USING THE REGULAR ONES | All contracts |
| [N-06] | USE SCIENTIFIC NOTATION (E.G. 1E18) RATHER THAN EXPONENTIATION (E.G. 10**18) | 11 |
| [N-07] | USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED() | 6 |
| [N-08] | NEED FUZZING TEST |  |
| [N-09] | NATSPEC IS INCOMPLETE | 6 |

| Total Non-Critical Issues | 9 | Total Instances | 17 |
|:--:|:--:|:--:|--:|

### Refactor Issues 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [R-01] | USE CUSTOM ERRORS INSTEAD OF REVERT STRINGS TO SAVE GAS | All contracts |
| [R-02] | DO NOT PRE-DECLARE VARIABLE WITH DEFAULT VALUES | 8 |
| [R-03] | SHORTHAND WAY TO WRITE IF/ELSE STATEMENT | 2 |

| Total Refactor Issues | 3 | Total Instances | 10 |
|:--:|:--:|:--:|--:|

### Ordinary Issues
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [O-01] | FUNCTION NAMING SUGGESTIONS | 4 |
| [O-02] | PROPER USE OF "GET" AS A FUNCTION NAME PREFIX | 6 |
| [O-03] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES (-= TOO) | 2 |

| Total Ordinary Issues | 3 | Total Instances | 12 |
|:--:|:--:|:--:|--:|

### Suggestion Details 
| Count | Explanation | 
|:--:|:-------|
| [S-01] | CONSIDER USING PAUSABLEUPGRADEABLE.SOL FROM OPENZEPPELIN | 1 |

| Total Suggestions | 1 |
|:--:|:--:|

# Detailed Findings

## [L-01] NO USE OF UPGRADEABLE IERC20
in the following contracts `WstEth.sol,SfrxEth.sol and Reth.sol` makes use of OpenZeppelin `OwnableUpgradeable.sol` in the file but does not use an upgradeable version of IERC20.sol

### MITIGATION
```diff
contracts/SafEth/derivatives/Reth.sol
- 6:  import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
+ 5:  import "@openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";
```

## [L-02] CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE
Handle ownership transfers with two steps and two transactions. First, allow the current owner to propose a new owner address. Second, allow the proposed owner (and only the proposed owner) to accept ownership, and update the contract owner internally

### PROOF OF CONCEPT
```solidity
contracts/SafEth/derivatives/WstEth.sol
34: _transferOwnership(_owner);

contracts/SafEth/derivatives/SfrxEth.sol
37:  _transferOwnership(_owner);

contracts/SafEth/SafEth.sol
53: _transferOwnership(msg.sender);

contracts/SafEth/derivatives/Reth.sol
34: _transferOwnership(_owner);
```
### MITIGATION
Implement zero address check and consider implementing a two step process where the owner nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.

## [L-03] MISSING EVENTS FOR ONLY FUNCTIONS THAT CHANGE CRITICAL PARAMETERS
The afunctions that change critical parameters should emit events. Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services. The alternative of directly querying on-chain contract state for such changes is not considered practical for most users/usages.

missing events and timelocks do not promote transparency and if such changes immediately affect users' perception of fairness or trustworthiness, they could exit the protocol causing a reduction in liquidity which could negatively impact protocol TVL and reputation.
### PROOF OF CONCEPT
```solidity
contracts/SafEth/derivatives/WstEth.sol
48: function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }

56: function withdraw(uint256 _amount) external onlyOwner {
        IWStETH(WST_ETH).unwrap(_amount);
        uint256 stEthBal = IERC20(STETH_TOKEN).balanceOf(address(this));
        IERC20(STETH_TOKEN).approve(LIDO_CRV_POOL, stEthBal);
        uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
        IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);
        // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
        require(sent, "Failed to send Ether");
    }    
```
Other instances in the following contracts :
`contracts/SafEth/derivatives/SfrxEth.sol`
- [Line of code in SfrxEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53)
- [Line of code in SfrxEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60-L88)
- [Line of code in Reth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58-L60)
- [Line of code in Reth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107-L114)

`There are multuples instances across the scope.`

### MITIGATION
Add Event-Emit

## [L-04] THE FUNCTIONS OF THE PROTOCOL ARE SERIOUSLY COMPROMISED
In the initialize functions, the address for the new owner is not verified to not be the zero address throughout the scope. This is because the `OwnableUpgradeable.sol` from OpenZeppelin does not perform this check, compromising other functions that rely on onlyOwner.

### PROOF OF CONCEPT
```solidity
the follow functions is from OpenZeppelin:
  
  /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`).
     * Internal function without access restriction.
     */
    function _transferOwnership(address newOwner) internal virtual {
        address oldOwner = _owner;
        _owner = newOwner;
        emit OwnershipTransferred(oldOwner, newOwner);
    }
```

The functions such as Withdraw, Deposit, SetMaxSlippage cannot be modified. In case of the contract owner mistakenly transferring ownership to address(0), we recommend considering the potential issue of omitting this check for the protocol.

### MITIGATION
Add a check that prevents the owner from being set to address(0).

## [L-05] IN THE EMITS, INCLUDE THE OLD AND NEW VALUES OF THE UPDATED PARAMETERS TO TRACK THE CHANGES MADE 
The following functions SafEth.setMaxSlippage, SafEth.setMinAmount, and SafEth.setMaxAmount make critical changes and should include the old and new values of the updated parameters so that users can be aware of the changes made.

### MITIGATION
The example of how to mitigate the problem we raised:
```diff
- 225:  emit ChangeMaxAmount(maxAmount);
+ 225:  emit ChangeMaxAmount(oldMaxAmount, newMaxAmount);
```


## [N-01] FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE
According to the Solidity style guide, functions should be laid out in the following order: 

- constructor() 
- receive() 
- fallback() 
- external
- public 
- internal 
- private

[Solidity Order Functions](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions)

Functions should be grouped according to their visibility and ordered: `within a grouping, place the view and pure functions last`

## [N-02] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS
There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

`constants.sol` Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

## [N-03] NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

### MITIGATION
```diff
staking/NeoTokyoStaker.sol#L1517-L1521
- 4:  pragma solidity ^0.8.0;
+ 4:  pragma solidity 0.8.0;
```

## [N-04] USE A MORE RECENT VERSION OF SOLIDITY
Old version of solidity is used, consider using the new one `0.8.19`. You can see what new versions offer regarding bug fixed [here](https://github.com/ethereum/solidity/blob/develop/Changelog.md)

Instances - All of the contracts.

### MITIGATION
```diff
staking/NeoTokyoStaker.sol#L1517-L1521
- 4:  pragma solidity 0.8.0;
+ 4:  pragma solidity 0.8.19;
```

## [N-05] CREATE YOUR OWN IMPORT NAMES INSTEAD OF USING THE REGULAR ONES
For better readability, you should name the imports instead of using the regular ones.
Example:
```solidity
5: import {DistributionTypes} from "../libraries/DistributionTypes.sol";
```
`Instances - All of the contracts.`

### MITIGATION
```diff
staking/NeoTokyoStaker.sol#L1517-L1521
- 6:  import "./IERC20.sol";
- 7:  import "./extensions/IERC20Metadata.sol";
- 8:  import "../../utils/Context.sol";
+ 6:  import {IERC20} from "./IERC20.sol";
+ 7:  import {IERC20Metadata} from "./extensions/IERC20Metadata.sol";
+ 8:  import {Context} from "../../utils/Context.sol";
```

## [N-06] USE SCIENTIFIC NOTATION (E.G. 1E18) RATHER THAN EXPONENTIATION (E.G. 10**18)
Use it for better code readability.

### PROOF OF CONCEPT
`contracts/SafEth/derivatives/WstEth.sol`
- [Line of code in WstEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L60)
- [Line of code in WstEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L87)

`contracts/SafEth/derivatives/SfrxEth.sol`
- [Line of code in SfrxEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75)
- [Line of code in SfrxEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L113-L115)

`contracts/SafEth/SafEth.sol`
- [Line of code in SafEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L80-L81)
- [Line of code in SafEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L94)
- [Line of code in SafEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L98)

`contracts/SafEth/derivatives/Reth.sol`
- [Line of code in Reth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L171)
- [Line of code in Reth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L173-L174)
- [Line of code in Reth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L215)


## [N-07] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED()
Since version 0.8.4 for appending bytes, [bytes.concat()](https://docs.soliditylang.org/en/v0.8.19/types.html#bytes-concat) can be used instead of abi.encodePacked().

### PROOF OF CONCEPT
```solidity
contracts/SafEth/derivatives/Reth.sol
69:  keccak256(abi.encodePacked("contract.address", "rocketTokenRETH"))
124: keccak256(abi.encodePacked("contract.address", "rocketDepositPool"))
135: keccak256(abi.encodePacked("contract.address","rocketDAOProtocolSettingsDeposit"))
161: keccak256(abi.encodePacked("contract.address", "rocketDepositPool"))
190: keccak256(abi.encodePacked("contract.address", "rocketTokenRETH"))
232: keccak256(abi.encodePacked("contract.address", "rocketTokenRETH"))
```

## [N-08] NEED FUZZING TEST
We recommend the use of fuzzing tests, especially in finance oriented protocols, due to the complexity and risk involved in handling large amounts of money in these smart contracts. Finance oriented contracts are critical in terms of security and accuracy, as any errors or vulnerabilities could be exploited by malicious attackers to steal funds or cause significant damage. 

Fuzzing tests are an important tool for identifying possible vulnerabilities in the code through the automatic and random generation of input data in the contract, which can help avoid costly errors in production. 

### RECOMMENDATION 
Use should fuzzing test like Echidna or [Foundry](https://book.getfoundry.sh/forge/fuzz-testing).

## [N-09] NATSPEC IS INCOMPLETE
It is important to have complete NatSpec in a project, especially in one that handles money, because it provides clear and detailed documentation of the contract, including its purpose, functions, variables, and overall structure. This documentation is essential for developers and users to understand the contract and use it effectively.

### PROOF OF CONCEPT
```solidity
contracts/SafEth/derivatives/WstEth.sol

/// @audit Missing: '@param _amount'
52: /**
        @notice - Owner only function to Convert derivative into ETH
        @dev - Owner is set to SafEth contract
     */
    function withdraw(uint256 _amount) external onlyOwner {

/// @audit Missing: '@return'
69: /**
        @notice - Owner only function to Deposit ETH into derivative
        @dev - Owner is set to SafEth contract
     */
    function deposit() external payable onlyOwner returns (uint256) {

/// @audit Missing: '@return'
83:  /**
        @notice - Get price of derivative in terms of ETH
     */
    function ethPerDerivative(uint256 _amount) public view returns (uint256){

/// @audit Missing: '@return'
90: /**
        @notice - Total derivative balance
     */
    function balance() public view returns (uint256) {

contracts/SafEth/derivatives/SfrxEth.sol

/// @audit Missing: '@return'
90:  /**
        @notice - Owner only function to Deposit into derivative
        @dev - Owner is set to SafEth contract
     */
    function deposit() external payable onlyOwner returns (uint256) {

/// @audit Missing: '@param _amount'
/// @audit Missing: '@return'
108: /**
        @notice - Get price of derivative in terms of ETH
     */
    function ethPerDerivative(uint256 _amount) public view returns (uint256) {

```

## [R-01] USE CUSTOM ERRORS INSTEAD OF REVERT STRINGS TO SAVE GAS
It is advisable to use custom errors instead of revert strings because they provide greater clarity and accuracy in the error information displayed in the transaction.

When using a revert string, only a natural language description of the error is provided, which can lead to misinterpretations and difficulties in debugging issues in the contract. Additionally, revert strings are more expensive in terms of gas, as they need to be encoded in the transaction.

On the other hand, custom errors can be defined to provide a specific numerical code that identifies the type of error, which can facilitate debugging issues. Additionally, custom errors are more gas-efficient, as only the numerical error code is sent instead of a complete string.

We found multiple instances with long revert strings. We recommend the use of custom errors in order to use and optimize your code in terms of gas and readability with the new improvements that Solidity offers.

## [R-02] DO NOT PRE-DECLARE VARIABLE WITH DEFAULT VALUES
If a variable is not set/initialized, it is assumed to have the default value (`0` for `uint`, `false` for `bool`, `address(0)` for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

example: 

Not recommended ❌:

```solidity
for (uint256 i = 0 ; i ...; i++) {// ...}
```
Recommended ✔: 

```solidity
for (uint256 i; i ...; i++) {// ...}
```
### PROOF OF CONCEPT
`contracts/SafEth/SafEth.sol`
- [Line of code in SafEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L68)
- [Line of code in SafEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71)
- [Line of code in SafEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L83-L84)
- [Line of code in SafEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113)
- [Line of code in SafEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140)
- [Line of code in SafEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147)
- [Line of code in SafEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L170-L171)
- [Line of code in SafEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L190-L191)

## [R-03] SHORTHAND WAY TO WRITE IF/ELSE STATEMENT
The regular if-else statement can be refactored in an abbreviated way to write it. This increases readability and reduces the overall SLOC (lines of code).

### PROOF OF CONCEPT
```solidity
contracts/SafEth/derivatives/Reth.sol

211: function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        if (poolCanDeposit(_amount))
            return
                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
        else return (poolPrice() * 10 ** 18) / (10 ** 18);
    }
```
`contracts/SafEth/SafEth.sol`
- [Line of code in SafEth ](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L79-L81)

## [O-01]  FUNCTION NAMING SUGGESTIONS
Proper use of _ as a function name prefix and a common pattern is to prefix internal and private function names with _. This pattern is correctly applied in all contracts, however there are some inconsistencies in just this contract.

### PROOF OF CONCEPT
```solidity
contracts/SafEth/derivatives/Reth.sol

83: function swapExactInputSingleHop(
        address _tokenIn,
        address _tokenOut,
        uint24 _poolFee,
        uint256 _amountIn,
        uint256 _minOut
    ) private returns (uint256 amountOut) {...}

120: function poolCanDeposit(uint256 _amount) private view returns (bool) {...}
228: function poolPrice() private view returns (uint256) {...}
666: function rethAddress() private view returns (address) {...}
``` 
### RECOMENDATION
```diff
staking/NeoTokyoStaker.sol#L1517-L1521
- 228:  function poolPrice() private view returns (uint256) {...}
+ 228:  function _poolPrice() private view returns (uint256) {...}
```

## [O-02] PROPER USE OF "GET" AS A FUNCTION NAME PREFIX
Clear function names can increase readability. Follow a standard convertion function names such as using `get` for getter `(view/pure) functions`.

### PROOF OF CONCEPT
```solidity
contracts/SafEth/derivatives/WstEth.sol

86: function ethPerDerivative(uint256 _amount) public view returns (uint256) {...}
93: function balance() public view returns (uint256) {...}

contracts/SafEth/derivatives/SfrxEth.sol

111: function ethPerDerivative(uint256 _amount) public view returns (uint256) {...}
122: function balance() public view returns (uint256) {...}

contracts/SafEth/derivatives/Reth.sol

211: function ethPerDerivative(uint256 _amount) public view returns (uint256) {...}
221:function balance() public view returns (uint256) {...}
```

### RECOMENDATION
```diff
staking/NeoTokyoStaker.sol#L1517-L1521
- 86:  ethPerDerivative(uint256 _amount) public view returns (uint256) {...}
+ 86:  getEthPerDerivative(uint256 _amount) public view returns (uint256) {...}
```

## [O-03]  <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES (-= TOO)
Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/catellaTech/8539f7345b17f0929a41598e7e00e3c2). Subtractions act the same way.

### PROOF OF CONCEPT
```solidity
contracts/SafEth/SafEth.sol

172: localTotalWeight += weights[i];
192: localTotalWeight += weights[i];
```
### RECOMENDATION
```diff
staking/NeoTokyoStaker.sol#L1517-L1521
- 172: localTotalWeight += weights[i];
+ 173: localTotalWeight = localTotalWeight + weights[i];
```
## [S-01] CONSIDER USING PAUSABLEUPGRADEABLE.SOL FROM OPENZEPPELIN
In the `SafEth.sol` contract, two functions `SafEth.etPauseStaking` and `SafEth.PauseUnstaking` are established. It is recommended to use [PausableUpgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/security/PausableUpgradeable.sol) from OpenZeppelin to reduce the SLOC of the smart contract, save gas and make the smart contract more legible.



