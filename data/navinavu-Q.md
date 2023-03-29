## Slippage parameter of setMaxSlippage() setter is not checked for non-zero value which might lead to potential DoS in staking and unstaking features

If an admin calls the `SafEth.setMaxSlippage()` function by mistakenly inputing 0 as `_slippage` parameter value, this might lead to DoS in the affected derivatives functions until the admin has updated the max slippage storage value of the affected derivative.

### Context
SafEth.sol:setMaxSlippage()

### PoC
##### Reth.sol:deposit()
The minimum swap amount is calculated as follow:
`uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) * ((10 ** 18 - maxSlippage))) / 10 ** 18);`
If `maxSlippage` is set to 0, this will lead to `minOut` being equal to `rethPerEth * msg.value / 10 ** 18`
which is equivalent to a swap without slippage that might revert the whole staking operation during the sub call to `ISwapRouter(UNISWAP_ROUTER).exactInputSingle(params);` as it can be seen at line `128` of the Uniswap v3 SwapRouter contract (https://github.com/Uniswap/v3-periphery/blob/main/contracts/SwapRouter.sol#L128).

##### WstEth.sol:withdraw()
The minimum swap amount is calculated as follow:
`uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;`
If `maxSlippage` is set to 0, this will lead to `minOut` being equal to `stEthBal / 10 ** 18`
which is equivalent to a swap without slippage that might revert the whole unstaking operation during the sub call to `IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);` as it can be seen at line `453` of the deployed `IStEthEthPool(LIDO_CRV_POOL)` external contract: (https://etherscan.io/address/0xDC24316b9AE028F1497c275EB9192a3Ea0f67022#code).

##### SfrxEth.sol:withdraw()
The minimum swap amount is calculated as follow:
`uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) * (10 ** 18 - maxSlippage)) / 10 ** 18;`
If `maxSlippage` is set to 0, this will lead to `minOut` being equal to `(ethPerDerivative(_amount) * _amount) / 10 ** 18`
which is equivalent to a swap without slippage that might revert the whole unstaking operation during the sub call to `IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).exchange(1,0,frxEthBalance,minOut);` as it can be seen at line `534` of the deployed `IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS)` external contract: (https://etherscan.io/address/0xa1F8A6807c402E4A15ef4EBa36528A3FED24E577#code).

### Mitigation
Assert that the `_maxSlippage` parameter value of the `SafEth.sol:setMaxSlippage()` function is non-zero.

---

## Misleading NATSPEC comment

Comment `@notice - Adds new derivative to the index fund` is misleading, this function is used to update the weight of a specific derivative.

### Context
SafEth.sol:adjustWeight() - NATSPEC

### Mitigation
Describe the valid role of the function: updating the weight of a specific derivative.

### Tools used
Manual review

---

## No event emitted on initialize() functions of implementations
There is no event emitted on the success of implementations' `initialize()` functions that might lead to a lack of transparency and difficult off-chain upgrades tracking.

### Context
Reth.sol:initialize()
WstEth.sol:initialize()
SfrxEth.sol:initialize()
SafEth.sol:initialize()

### Mitigation
Emit an event on initialization.

### Tools used
Manual review

---

## Events should be emitted before external calls to unknown addresses
In respect to the check-effect-interaction pattern, the events should be emitted before any external calls to uncontrolled addresses to avoid issues for off-chain third-parties in the scenario of an eventual reentering call.

### Context
Global

### POC
Example in `SafEth.unstake()`:
```solidity
function unstake(uint256 _safEthAmount) external {
	// ... (code removed for clarity)
		
	(bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
		""
	);
	require(sent, "Failed to send Ether");
	// Emits after the external call to an uncontrolled address that might reenter
	emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
}
```

### Mitigation
Emit events before external calls on uncontrolled addresses when possible.

Example in `SafEth.unstake()`:

```solidity
function unstake(uint256 _safEthAmount) external {
	// ... (code removed for clarity)
		
	// Emits before the external call to uncontrolled address
	emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
		
	(bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
	    ""
	);
	require(sent, "Failed to send Ether");
	//emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
}
```

### Tools used
Manual review

---

## Shadowing of local variable name with inherited function name
The `totalSupply` local value of the `stake()` function shadows the inherited `totalSupply()` function name. 
In this particular context, it does not represent a threat to the security of the contract but could potentially introduce vulnerabilities as the protocol/contract grows (upgrades). 
This also leads to less readability and potential developer errors.

### Context
SafEth.sol:stake()

### POC
```solidity
function stake() external payable {
    // ... (code removed for clarity)
		
    // totalSupply shadows inherited totalSupply() function name
    uint256 totalSupply = totalSupply();
    uint256 preDepositPrice;
    if (totalSupply == 0)
        preDepositPrice = 10 ** 18;
    else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

    // ... (code removed for clarity)
}
```

### Mitigation
Use a different local variable name.

### Tools used
Manual review
