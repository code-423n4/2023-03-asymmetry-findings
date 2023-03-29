## [NC-1] Import declarations should import specific identifiers, rather than the whole file
Using import declarations of the form `import {<identifier_name>} from "My/Contract.sol"` avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation. Avoid import declarations of the form `import "My/Contract.sol;`.
Files: All files.

Consider using named imports

Files: All files.

## [NC-2] INCOMPLETE NATSPEC COMMENTS
The NatSpec comments on the does not include the input field. Consider adding the amount argument to Natspec definition.
Files:
- [contracts/SafEth/derivatives/Reth.sol#L105](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L105)
- [contracts/SafEth/derivatives/WstEth.sol#L86](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L86)
- [contracts/SafEth/derivatives/WstEth.sol#L56](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56)

```
/**
        @notice - Convert derivative into ETH
     */
    function withdraw(uint256 amount) external onlyOwner {
        RocketTokenRETHInterface(rethAddress()).burn(amount);
        // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
        require(sent, "Failed to send Ether");
    }

    /**
        @notice - Get price of derivative in terms of ETH
     */
    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
            10 ** 18
        );
        return ((10 ** 18 * frxAmount) /
            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
    }

```

## [NC-3] USE LATEST SOLIDITY PRAGMA VERSION
Consider using Solidity latest stable pragma version 0.8.19. 
Latest stable software have bug fixes and security updates.
See: https://swcregistry.io/docs/SWC-102
Files: All Files

## [NC-4] AVOID FLOATING PRAGMA
Avoid the use of floating pragma and use locked pragma
See: https://swcregistry.io/docs/SWC-103
Files: All files.

Use 
```
pragma solidity 0.8.19;
```
instead of 
```
pragma solidity ^0.8.13;
```

## [NC-4] Remove unused argument or add comments as to why it is not used.
The `_amount` argument is not used in the `ethPerDerivative` function of `SafEth/derivative/SfrxEth.sol` file.

Consider removing the unused function or provide a comment why it is not used.

File: [contracts/SafEth/derivatives/SfrxEth.sol#L111](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L111)
```
    /**
        @notice - Get price of derivative in terms of ETH
     */
    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
            10 ** 18
        );
        return ((10 ** 18 * frxAmount) /
            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
    }

```

## [NC-5] USE LATEST OPENZEPPELIN CONTRACTS.
The project uses the openzeppelin/contract version 4.8.0 while the latest version is 4.8.2. Consider using the latest version of openzeppelin/contract as newer version will have security fixes and patches.

File: package.json

## [NC-6] EMIT EVENT FOR USEFUL STATE UPDATES
The `setMaxSlippage` function in the `Reth.sol` file does not emit an event when maximum slippage is set. Emitted events are monitored by external systems.

- [contracts/SafEth/derivatives/Reth.sol#L58](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L58)
```
File: contracts/SafEth/derivatives/Reth.sol#L58
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```
## [NC-7] Upgradable contract is missing a --gap[50] storage variables to allow for new storage variables in future upgrades.

see: https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps

Files: All files
Consider adding the Storage Gaps as recommended by openzeppelin.

## [NC-8] Events that mark critical parameter changes should contain both the old and the new value
Files: 
- [contracts/SafEth/SafEth.sol#L223](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223)
- [contracts/SafEth/SafEth.sol#L214](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L214)

## [NC‑9] Contracts should have full test coverage
Although 100% test coverage is not an indication of lack of bugs, it will aid in catching some easy bugs and also help in audit.
Files: 
- contracts/SafEth/derivatives/SfrxEth.sol 
- contracts/SafEth/derivatives/WstEth.sol 
- contracts/SafEth/derivatives/Reth.sol

## [NC‑10] Function ordering does not follow the Solidity style guide
Solidity docs suggests the functions should be laid out in this order
 :constructor(), receive(), fallback(), external, public, internal, private.
However all the files do not follow this pattern.

Files: All files.

## [L-01] Solidity Optimizer Be Problematic

The Solidity optimizer is enabled in the project's `hardhat.config.ts` file. 
see: https://blog.soliditylang.org/2017/05/03/solidity-optimizer-bug/
```
const config: HardhatUserConfig = {
  solidity: {
    version: "0.8.13",
    settings: {
      optimizer: {
        enabled: true,
        runs: 100000,
      },
```
## [L-02] The ` __Ownable_init()` function is not invoked in the initialize functions of all the files.

All Smart contract files inherit from openzeppelin's OwnableUpgradable but the ` __Ownable_init()` function of the OwnableUpgradable contract is not called in the init functions of the smart contracts instead, the `transferFrom` function is called.

See: https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable

> "Another difference between a constructor and a regular function is that Solidity takes care of automatically invoking the constructors of all ancestors of a contract. When writing an initializer, you need to take special care to manually call the initializers of all parent contracts. Note that the initializer modifier can only be called once even when using inheritance, so parent contracts should use the onlyInitializing modifier:"

Files: initialize functions in all files.

```
function initialize(string memory _tokenName, string memory _tokenSymbol) external initializer {
        ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
        _transferOwnership(msg.sender);
        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
    }
```
#### Recommended Mitigation steps
Consider calling all the parent initialize function (` __Ownable_init()`) in the smart contracts and remove the `transferOwnership` function call.

## [L-03] Consider using `Ownable2StepUpgradable` in place of the `OwnableUpgradable`.

The smart contracts currently lack 2 step ownership change and can lead to the irrecoverable setting of a new owner. The 2 step ownership change allow for the newOwner to confirm the change and the process can be reversed if a wrong address is set with the `transferOwnership` function.

```
/**
     * @dev Starts the ownership transfer of the contract to a new account. Replaces the pending transfer if there is one.
     * Can only be called by the current owner.
     */
    function transferOwnership(address newOwner) public virtual override onlyOwner {
        _pendingOwner = newOwner;
        emit OwnershipTransferStarted(owner(), newOwner);
    }

    /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`) and deletes any pending owner.
     * Internal function without access restriction.
     */
    function _transferOwnership(address newOwner) internal virtual override {
        delete _pendingOwner;
        super._transferOwnership(newOwner);
    }

    /**
     * @dev The new owner accepts the ownership transfer.
     */
    function acceptOwnership() external {
        address sender = _msgSender();
        require(pendingOwner() == sender, "Ownable2Step: caller is not the new owner");
        _transferOwnership(sender);
    }

```

