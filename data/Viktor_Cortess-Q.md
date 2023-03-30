### [L-01] Derivative contracts could be initialized with 0 address owner

There is no checking of 0 address, so this code from the Test showing deployment of derivative contracts works:

     describe("Derivatives", async () => {
      let derivatives = [] as any;
      beforeEach(async () => {
      await resetToBlock(initialHardhatBlock);
      derivatives = [];
      const factory0 = await ethers.getContractFactory("Reth");
      const factory1 = await ethers.getContractFactory("SfrxEth");
      const factory2 = await ethers.getContractFactory("WstEth");

      const derivative0 = await upgrades.deployProxy(factory0, [
        "0x0000000000000000000000000000000000000000",
      ]);
      await derivative0.deployed();

This could lead to redeployment and reinitialization.

### [L-02] Accepting ownership makes the process of transferring ownership safer

Recommend considering implementing a two-step process where the owner or admin nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.

### [N-01 ] INCONSISTENT SOLIDITY VERSIONS

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

All files: pragma solidity ^0.8.13;

### [N-02] NON-USAGE OF SPECIFIC IMPORTS

All files.

It’s possible to name the imports to improve code readability