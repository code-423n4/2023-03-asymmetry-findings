# Lines of code

## Unverified input:

### SafEth:

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202-L208

### Deriviatives:

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48-L50
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L58-L60

## Resulting problem locations:

### Deposit/Stake:

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L173-L174

### Withdraw/Unstake:

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L60
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75

# Vulnerability detail

## Impact

`maxSlippage` parameter can be set to an arbitrary `uint256` without any additional verification through `setMaxSlippage` function in `SafEth`. In current derivative contracts `maxSlippage` is subtracted from $10^{18}$ representing 100% in places linked in "Resulting problem locations":
`(10 ** 18 - maxSlippage)`

If `maxSlippage` is set to a value bigger than $10^{18}$ this will revert `stake` and/or `unstake` transactions due to Solidity's arithmetic error, effectively pausing `stake` and/or `unstake` functions, depending on which derivative's `maxSlippage` is set over 100%.

## Proof of Concept

```
// The following test passes
it("blocks unstaking in case of slippage > 100%", async function () {
	const derivativeCount = (await safEthProxy.derivativeCount()).toNumber();
	for (let i = 0; i < derivativeCount; i++) {
		await safEthProxy.setMaxSlippage(i, ethers.utils.parseEther("10")); // 1000% slippage
	}

	const safEthBalance = await safEthProxy.balanceOf(adminAccount.address);
	await expect(safEthProxy.unstake(safEthBalance)).to.be.reverted;
});
```

## Problem description

`setMaxSlippage` is protected by `onlyOwner` modifier and it is expected from the **owner** of the `SafEth` contract to be acting in the best interest of the protocol. This being said, while setting the slippage by the **owner** anywhere within the range of 0-100% *might* be seen as a potentially harmful/bad decision, setting the slippage above 100%, considering the consequences described in the previous section is either a mistake or an intentional pause of the protocol.

While there is nothing implicitly wrong with pausing the protocol and the `SafEth` contract implements pausing functionality for both `stake` and `unstake` functions, exposing it as well through `setMaxSlippage` *may* be seen as *sneaky* and, in case of such mistake actually happening, potentially severely harm protocols credibility.

## Recommended mitigation

Verify `uint _slippage` input parameter in `setMaxSlippage` by adding the following line (or similar) before setting the slippage:

```
reuire(_slippage <= 10 ** 18, "slippage too high");
```