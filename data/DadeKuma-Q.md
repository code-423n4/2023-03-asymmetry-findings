
## Summary

---

### Low Risk Findings
|Id|Title|Instances|
|:--:|:-------|:--:|
|[L-01]| No limits to max slippage | 1 |
|[L-02]| Missing safe limits in `setMinAmount` and `setMaxAmount` | 1 |
|[L-03]| Owner can renounce ownership | 17 |
|[L-04]| Lack of two-step Ownership | 4 |

Total: 23 instances over 4 issues.

### Non Critical Findings
|Id|Title|Instances|
|:--:|:-------|:--:|
|[NC-01]| Static `keccak256` computed more than once | 1 |
|[NC-02]| Parameter omission in events | 1 |
|[NC-03]| Some functions don't follow the Solidity naming conventions | 4 |
|[NC-04]| Use of floating pragma | 27 |
|[NC-05]| Missing or incomplete NatSpec | 7 |

Total: 40 instances over 5 issues.

## Low Risk Findings

---

### [L-01] No limits to max slippage


A slippage set to 100% could result in massive loss of funds due to frontrunning. Consider setting a max limit to avoid this risk.


*There is 1 instance of this issue.*


```solidity
File: contracts/SafEth/SafEth.sol

202: 		    function setMaxSlippage(
203: 		        uint _derivativeIndex,
204: 		        uint _slippage
205: 		    ) external onlyOwner {
206: 		        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
207: 		        emit SetMaxSlippage(_derivativeIndex, _slippage);
208: 		    }

```

### [L-02] Missing safe limits in `setMinAmount` and `setMaxAmount`


The `minAmount` should never be greater than `maxAmount`, as it would lock the `stake` function. Consider adding some safechecks to those setters.


*There is 1 instance of this issue.*


```solidity
File: contracts/SafEth/SafEth.sol

214: 		    function setMinAmount(uint256 _minAmount) external onlyOwner {
215: 		        minAmount = _minAmount;
216: 		        emit ChangeMinAmount(minAmount);
217: 		    }
223: 		    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
224: 		        maxAmount = _maxAmount;
225: 		        emit ChangeMaxAmount(maxAmount);
226: 		    }

```

### [L-03] Owner can renounce ownership

The contract's owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

Openzeppelin's `Ownable` used in this project implements `renounceOwnership`. This can represent a certain risk if the ownership is renounced for any other reason than by design.

Renouncing ownership will leave the contract without an Owner, therefore removing any functionality that needs authority.

*There are 17 instances of this issue.*


```solidity
File: contracts/SafEth/SafEth.sol

138: 		    function rebalanceToWeights() external onlyOwner {

168: 		    ) external onlyOwner {

185: 		    ) external onlyOwner {

205: 		    ) external onlyOwner {

214: 		    function setMinAmount(uint256 _minAmount) external onlyOwner {

223: 		    function setMaxAmount(uint256 _maxAmount) external onlyOwner {

232: 		    function setPauseStaking(bool _pause) external onlyOwner {

241: 		    function setPauseUnstaking(bool _pause) external onlyOwner {


File: contracts/SafEth/derivatives/Reth.sol

58: 		    function setMaxSlippage(uint256 _slippage) external onlyOwner {

107: 		    function withdraw(uint256 amount) external onlyOwner {

156: 		    function deposit() external payable onlyOwner returns (uint256) {


File: contracts/SafEth/derivatives/SfrxEth.sol

51: 		    function setMaxSlippage(uint256 _slippage) external onlyOwner {

60: 		    function withdraw(uint256 _amount) external onlyOwner {

94: 		    function deposit() external payable onlyOwner returns (uint256) {


File: contracts/SafEth/derivatives/WstEth.sol

48: 		    function setMaxSlippage(uint256 _slippage) external onlyOwner {

56: 		    function withdraw(uint256 _amount) external onlyOwner {

73: 		    function deposit() external payable onlyOwner returns (uint256) {

```

### [L-04] Lack of two-step Ownership

A single step Ownership change is risky due to the fact that the address could be wrong, so it's better to use a two - step Ownership.

*There are 4 instances of this issue.*


```solidity
File: contracts/SafEth/SafEth.sol

9: 		import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";


File: contracts/SafEth/derivatives/Reth.sol

13: 		import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";


File: contracts/SafEth/derivatives/SfrxEth.sol

6: 		import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";


File: contracts/SafEth/derivatives/WstEth.sol

5: 		import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

```

## Non Critical Findings

---

### [NC-01] Static `keccak256` computed more than once


`keccak256` computation produces the same result as there aren't any dynamic components: there's no need to recompute it for each function call. Consider refactoring them as constants to save gas.


*There is 1 instance of this issue.*


```solidity
File: contracts/SafEth/derivatives/Reth.sol

69: 		                keccak256(
70: 		                    abi.encodePacked("contract.address", "rocketTokenRETH")
71: 		                )
124: 		                keccak256(
125: 		                    abi.encodePacked("contract.address", "rocketDepositPool")
126: 		                )
135: 		                keccak256(
136: 		                    abi.encodePacked(
137: 		                        "contract.address",
138: 		                        "rocketDAOProtocolSettingsDeposit"
139: 		                    )
161: 		                keccak256(
162: 		                    abi.encodePacked("contract.address", "rocketDepositPool")
163: 		                )
190: 		                    keccak256(
191: 		                        abi.encodePacked("contract.address", "rocketTokenRETH")
192: 		                    )
232: 		                keccak256(
233: 		                    abi.encodePacked("contract.address", "rocketTokenRETH")
234: 		                )

```

### [NC-02] Parameter omission in events

Events are generally emitted when sensitive changes are made to the contracts.

However, some are missing important parameters, as they should include both the new value and old value where possible.

*There is 1 instance of this issue.*


```solidity
File: contracts/SafEth/SafEth.sol

207: 		        emit SetMaxSlippage(_derivativeIndex, _slippage);

```

### [NC-03] Some functions don't follow the Solidity naming conventions

Follow the Solidity naming convention by adding an underscore before the function name, for any `private` and `internal` functions.

*There are 4 instances of this issue.*


```solidity
File: contracts/SafEth/derivatives/Reth.sol

66: 		    function rethAddress() private view returns (address) {

83: 		    function swapExactInputSingleHop(

120: 		    function poolCanDeposit(uint256 _amount) private view returns (bool) {

228: 		    function poolPrice() private view returns (uint256) {

```

### [NC-04] Use of floating pragma

Locking the pragma helps avoid accidental deploys with an outdated compiler version that may introduce bugs and unexpected vulnerabilities.

Floating pragma is meant to be used for libraries and contracts that have external users and need backward compatibility.

*There are 27 instances of this issue.*

<details>
<summary>Expand findings</summary>


```solidity
File: contracts/SafEth/SafEth.sol

2: 		pragma solidity ^0.8.13;


File: contracts/SafEth/derivatives/Reth.sol

2: 		pragma solidity ^0.8.13;


File: contracts/SafEth/derivatives/SfrxEth.sol

2: 		pragma solidity ^0.8.13;


File: contracts/SafEth/derivatives/WstEth.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/curve/IAfEthPool.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/curve/ICrvEthPool.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/curve/ICrvEthPoolLegacy.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/curve/IFrxEthEthPool.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/curve/IFxsEthPool.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/curve/IStEthEthPool.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/frax/IFrxETHMinter.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/frax/IsFrxEth.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/lido/IWStETH.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/lido/IstETH.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/rocketpool/RocketDepositPoolInterface.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/rocketpool/RocketStorageInterface.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/rocketpool/RocketTokenRETHInterface.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/uniswap/ISwapRouter.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/uniswap/IUniswapV3Factory.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/uniswap/IUniswapV3Pool.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/uniswap/pool/IUniswapV3PoolActions.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/uniswap/pool/IUniswapV3PoolDerivedState.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/uniswap/pool/IUniswapV3PoolEvents.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/uniswap/pool/IUniswapV3PoolImmutables.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/uniswap/pool/IUniswapV3PoolOwnerActions.sol

2: 		pragma solidity ^0.8.13;


File: contracts/interfaces/uniswap/pool/IUniswapV3PoolState.sol

2: 		pragma solidity ^0.8.13;

```
</details>

---
### [NC-05] Missing or incomplete NatSpec

Some functions miss a NatSpec, or they have an incomplete one.
It is recommended that contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI).

Include either `@notice` and/or `@dev` to describe how the function is supposed to work, a `@param` to describe each parameter, and a `@return` for any return values.

*There are 7 instances of this issue.*


```solidity
File: contracts/SafEth/derivatives/Reth.sol

104: 		    /**
105: 		        @notice - Convert derivative into ETH
106: 		     */
107: 		    function withdraw(uint256 amount) external onlyOwner {

152: 		    /**
153: 		        @notice - Deposit into derivative
154: 		        @dev - will either get rETH on exchange or deposit into contract depending on availability
155: 		     */
156: 		    function deposit() external payable onlyOwner returns (uint256) {


File: contracts/SafEth/derivatives/SfrxEth.sol

48: 		    /**
49: 		        @notice - Owner only function to set max slippage for derivative
50: 		    */
51: 		    function setMaxSlippage(uint256 _slippage) external onlyOwner {

90: 		    /**
91: 		        @notice - Owner only function to Deposit into derivative
92: 		        @dev - Owner is set to SafEth contract
93: 		     */
94: 		    function deposit() external payable onlyOwner returns (uint256) {


File: contracts/SafEth/derivatives/WstEth.sol

45: 		    /**
46: 		        @notice - Owner only function to set max slippage for derivative
47: 		    */
48: 		    function setMaxSlippage(uint256 _slippage) external onlyOwner {

52: 		    /**
53: 		        @notice - Owner only function to Convert derivative into ETH
54: 		        @dev - Owner is set to SafEth contract
55: 		     */
56: 		    function withdraw(uint256 _amount) external onlyOwner {

69: 		    /**
70: 		        @notice - Owner only function to Deposit ETH into derivative
71: 		        @dev - Owner is set to SafEth contract
72: 		     */
73: 		    function deposit() external payable onlyOwner returns (uint256) {

```
