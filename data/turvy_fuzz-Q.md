### L - Owner can renounce Ownership
**Description:**
The Asymmetry Ownable used in this project contract implements renounceOwnership.
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L9
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L13
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L6
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L5

This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner and there are a lot of critical functions that implements the onlyOwner

**Recommendation:**
We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.

### NC - Add a Timelock to Critical Parameter Change
It is a good practice to give time for users to react and adjust to critical changes with a mandatory time window between them. The first step merely broadcasts to users that a particular change is coming, and the second step commits that change after a suitable waiting period. This allows users that do not accept the change to withdraw within the grace period. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious Owner making any malicious or ulterior intention). Specifically, privileged roles could use front running to make malicious changes just ahead of incoming transactions, or purely accidental negative effects could occur due to the unfortunate timing of changes.

Consider extending the timelock feature beyond contract ownership management to business critical functions. Here are some of the instances entailed:

**setMaxSlippage - https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202**
**setMinAmount - https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214**
**setMaxAmount - https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223**

### NC - High Centralization (Contract Owner Has Too Many Privileges)
The owner of the contracts has too many privileges relative to standard users which is not inline with the business logic - A protocol to help diversify and decentralize liquid staking derivatives. The consequence is disastrous if the contract owner’s private key has been compromised. And, in the event the key was lost or unrecoverable, no implementation upgrades and system parameter updates will ever be possible.

For a project this grand, it increases the likelihood that the owner will be targeted by an attacker, especially given the insufficient protection on sensitive owner private keys. The concentration of privileges creates a **single point of failure**; and possibly result if the ownership is compromised will be: **hijacking the entire protocol**.

Consider:

**1. splitting privileges (e.g. via the multisig option) to ensure that no one address has excessive ownership of the system**
**2. clearly documenting the functions and implementations the owner can change**
**3. documenting the risks associated with privileged users and single points of failure, and**
**4. ensuring that users are aware of all the risks associated with the system**

### NC - Omissions in Events
Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

Events with no old value;
**https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L174**
**https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L216**
**https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L225**

### NC - Lock pragmas to specific compiler version
**Description:**
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.
**https://swcregistry.io/docs/SWC-103**

**Recommendation:**
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.
**[locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)**

### NC - Empty/Unused Function Parameters
Empty or unused function parameters should be commented out as a better and declarative way to silence runtime warning messages. As an example, the following function may have these parameters refactored to:
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L111
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86

### NC - Include return parameters in NatSpec comments

**Description:**
If Return parameters are declared, you must prefix them with /// @return.

**https://docs.soliditylang.org/en/v0.8.15/natspec-format.html**

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the “@return” tag, they do incomplete analysis.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L89

There are 11 instance of this issue

**Recommendation:**
Include return parameters in NatSpec comments

### NC - Empty blocks should be removed or Emit something
**Description:**
Code contains empty block

*contracts/SafEth/SafEth.sol:*
  **246:     receive() external payable {}**

*contracts/SafEth/derivatives/SfrxEth.sol:*
  **126:     receive() external payable {}**

*contracts/SafEth/derivatives/Reth.sol:*
  **244:     receive() external payable {}**

*contracts/SafEth/derivatives/WstEth.sol:*
  **97:     receive() external payable {}**

**Recommendation:**
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

### NC - Use a more recent version of Solidity
**Context:**
All contracts

**Description:**
For security, it is best practice to use the latest Solidity version.
For the security fix list in the versions;
**https://github.com/ethereum/solidity/blob/develop/Changelog.md**

*Recommendation:*
Old version of Solidity is used `(0.8.13)`, newer version can be used `(0.8.17)`