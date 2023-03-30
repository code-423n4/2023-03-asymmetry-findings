## Low Risk Issues List
### Issue 
## [L-01] Use Ownable2StepUpgradeable instead of OwnableUpgradeable contract (04 instances)
## [L-02] Hardcode the address causes no future updates (01 instances)
## [L-03] initialize() function can be called by anybody
### Total:  03 issues
## [L-01] Use Ownable2StepUpgradeable instead of OwnableUpgradeable contract (04 instances)
[WstEth.sol#L5](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L5)
[SfrxEth.sol#L6](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L6)
[SafEth.sol#L9](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L9)
[Reth.sol#L13](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L13)
transferOwnership function is used to change Ownership from OwnableUpgradeable.sol.

There is another Openzeppelin Ownable contract (Ownable2StepUpgradeable.sol) has transferOwnership function, use is more secure due to 2-stage ownership transfer.
[Ownable2StepUpgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol)
 
## [L-02] Hardcode the address causes no future updates (01 instances)
```
contracts/SafEth/derivatives/Reth.sol
24:    address public constant UNISWAP_ROUTER =
25:        0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45;
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L24-L25
ROUTER etc. In case the addresses change due to reasons such as updating their versions in the future, addresses coded as constants cannot be updated, so it is recommended to add the update option with the onlyOwner modifier.

## [L-03] initialize() function can be called by anybody
### Description: 
initialize() function can be called by anybody when the contract is not initialized.

More importantly, if someone else runs this function, they will have full authority because of the _transferOwnership() function.

Here is a definition of initialize() function.
```
File: contracts/SafEth/derivatives/WstEth.sol
    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }

File: contracts/SafEth/derivatives/SfrxEth.sol
    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }

File: contracts/SafEth/SafEth.sol 
    function initialize(
        string memory _tokenName,
        string memory _tokenSymbol
    ) external initializer {
        ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
        _transferOwnership(msg.sender);
        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
    }

File: contracts/SafEth/derivatives/Reth.sol 
    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }
```
### Recommended Mitigation Steps
Add a control that makes initialize() only call the Deployer Contract or EOA;
```
if (msg.sender != DEPLOYER_ADDRESS) {
            revert NotDeployer();
        }
```

## Non-Critical Issues List
### Issue 
## [N-01] Add parameter to Event-Emit (01 instances)
## [N-02] Use a more recent version of Solidity (All Contracts)
## [N-03] Solidity compiler optimizations can be problematic (01 instances)
## [N-04] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS (All Contracts)
## [N-05] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES (All Contracts)
## [N-06] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,) (06 instances)
## [N-07] Function order (04 instances)
## [N-08] Lock pragmas to specific compiler version (All Contracts)
## [N-09] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS
## [N-10] MISSING EVENT FOR CRITICAL PARAMETERS INIT AND CHANGE (09 instances)
### Total:  10 issues
## [N-01] Add parameter to Event-Emit (01 instances)
Some event-emit description hasn’t parameter. Add to parameter for front-end website or client app , they can has that something has happened on the blockchain.
Events with no old value;
```
File: contracts/SafEth/SafEth.sol
154:         emit Rebalanced();
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L154

## [N-02] Use a more recent version of Solidity (All Contracts)
### Description: For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

## [N-03] Solidity compiler optimizations can be problematic (01 instances)
```
File: hardhat.config.ts
26:    settings: {
27:      optimizer: {
28:        enabled: true,
29:        runs: 100000,
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/hardhat.config.ts#L26-L29
### Description: Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

Exploit Scenario A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

### Recommendation: Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug. Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## [N-04] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS (All Contracts)
### Description:
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html
### Recommendation:
NatSpec comments should be increased in contracts

## [N-05] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES (All Contracts)
### Description:
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.
### Recommendation:
import {contract1 , contract2} from "filename.sol";

## [N-06] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,) (06 instances)
### Description:
Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled
Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,).
```
File: contracts/SafEth/derivatives/Reth.sol
70:                    abi.encodePacked("contract.address", "rocketTokenRETH")
125:                    abi.encodePacked("contract.address", "rocketDepositPool")
136:                    abi.encodePacked(
162:                    abi.encodePacked("contract.address", "rocketDepositPool")
191:                        abi.encodePacked("contract.address", "rocketTokenRETH")
233:                    abi.encodePacked("contract.address", "rocketTokenRETH")
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L70
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L125
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L136
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L162
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L191
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L233

## [N-07] Function order (04 instances)
Functions should be ordered following the Soldiity conventions ( https://docs.soliditylang.org/en/v0.8.15/style-guide.html#order-of-functions): receive() function should be placed after the constructor and before every other function.
```
File: contracts/SafEth/derivatives/WstEth.sol
97:    receive() external payable {}
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L97

```
File: contracts/SafEth/derivatives/SfrxEth.sol
126:    receive() external payable {}
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L126

```
File: contracts/SafEth/SafEth.sol
246:    receive() external payable {}
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L246

```
File: contracts/SafEth/derivatives/Reth.sol
244:    receive() external payable {}
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L244

## [N-08] Lock pragmas to specific compiler version (All Contracts)
### Description: Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally. https://swcregistry.io/docs/SWC-103

### Recommendation: Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version. 
https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/

## [N-09] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS
There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values):

This will help with readability and easier maintenance for future changes.
constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution

## [N-10] MISSING EVENT FOR CRITICAL PARAMETERS INIT AND CHANGE (09 instances)
### Description:
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
### Recommendation:
Add Event-Emit
```
File: contracts/SafEth/derivatives/WstEth.sol
48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
56:    function withdraw(uint256 _amount) external onlyOwner {
73:    function deposit() external payable onlyOwner returns (uint256) {
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L73

```
File: contracts/SafEth/derivatives/SfrxEth.sol
51:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
60:    function withdraw(uint256 _amount) external onlyOwner {
94:    function deposit() external payable onlyOwner returns (uint256) {
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L60
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L94

```
File: contracts/SafEth/derivatives/Reth.sol
58:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
107:    function withdraw(uint256 amount) external onlyOwner {
156:    function deposit() external payable onlyOwner returns (uint256) {
```
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L58
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L107
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L156

## Suggestions:
### Issue
## [S-01] GENERATE PERFECT CODE HEADERS EVERY TIME
## [S-02] Include the project name and development team information in the contract to increase the popularity and trust of users in the project
### Total: 02  issues
## [S-01] GENERATE PERFECT CODE HEADERS EVERY TIME
### Description:
I recommend using header for Solidity code layout and readability
https://github.com/transmissions11/headers
```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## [S-02] Include the project name and development team information in the contract to increase the popularity and trust of users in the project
### Recommendation: Use form like FraxFinance project
https://github.com/FraxFinance/frxETH-public/blob/7f7731dbc93154131aba6e741b6116da05b25662/src/sfrxETH.sol#L4-L24