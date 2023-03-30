# [L-01] Outdated Compiler 0.8.13 Used by the Contracts

#### Description:

During the analysis it was observed that the contracts code written in Solidity uses an outdated compiler version 0.8.13. The use of an outdated compiler version 0.8.13 poses several security risks that can compromise the security and stability of the smart contract. The following are the key findings of our audit:

**Code Generation:** Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.

Apart from these, there are several minor bug fixes and improvements.

- Assembler: Avoid duplicating subassembly bytecode where possible.
- Code Generator: Avoid including references to the deployed label of referenced functions if they are called right away.
- ContractLevelChecker: Properly distinguish the case of missing base constructor arguments from having an unimplemented base function.
- SMTChecker: Fix internal error caused by unhandled `z3` expressions that come from the solver when bitwise operators are used.
- SMTChecker: Fix internal error when using the custom NatSpec annotation to abstract free functions.
- TypeChecker: Also allow external library functions in `using for`.

#### Steps to reproduce:

Navigate to the following contracts and directory: 

`https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/interfaces/IDerivative.sol#L2`

````

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;
````

`https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts](https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts)/*`

#### Mitigation

The minimum required version must be [0.8.19](https://github.com/ethereum/solidity/releases/tag/v0.8.19). I strongly recommend that  update the compiler version to the latest version available. This will help to ensure the security and functionality of the contract.

The latest version of the Solidity compiler contains significant improvements and bug fixes, and is also more secure than the previous versions. Updating the compiler version will help to ensure that the contract remains compatible with future updates to the Solidity language.



# [L-02] Use _safeMint instead of _mint

#### Description
During the analysis it was observed that the contract is using function `_mint()` instead of _safeMint(). _mint() has the potential to allow an attacker to mint an arbitrary number of tokens, which could result in a supply inflation vulnerability. Additionally, if the contract does not have proper checks and balances in place, _mint() could be used to generate tokens and transfer them to arbitrary addresses, including attacker-controlled addresses.

##### Steps to reproduce
Navigate to following URL: https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L99
````
99        _mint(msg.sender, mintAmount);

````

##### Recommendations:
To address these potential vulnerabilities, we recommend using the alternative function, _safeMint(). _safeMint() offers additional security guarantees by ensuring that the number of tokens minted is always within a specified range, and that tokens are only transferred to addresses that have been properly vetted and approved by the contract.
Based on our analysis of the smart contract, we recommend the following changes to improve its security:
1- Replace _mint() with _safeMint() to ensure that tokens are only minted and transferred to authorized addresses.
2- Implement proper checks and balances to ensure that only authorized users are able to call _safeMint().
3- Consider adding additional security measures, such as multi-factor authentication or permission-based access controls, to further restrict access to the _safeMint() function.




# [L-03] SafEth.sol contracts utilised the draft stage  OwnableUpgradeable.sol and ERC20PermitUpgradeable.sol contracts

### Description

The `SafEth.sol` contracts utilised `OwnableUpgradeable.sol` and `ERC20PermitUpgradeable.sol`, an OpenZeppelin contracts. These contracts are still in draft stage and are not considered ready for mainnet use, thus are subject to potential changes in the future.

#### Proof of Concept:

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L9](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L9) 

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L9)11 

```solidity
9   import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

11  import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

```
