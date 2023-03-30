## Summary

Risk | Title
---|---
L01 | Bad english in comment in `Reth.sol`
L02 | Missing the `@return` NatSpec tag for some functions in `Reth.sol`
L03 | Misleading and wrong `@notice` NatSpec tag for the function `SafEth.adjustWeight`
L04 | `maxSlippage` can be unfavorable for the user and can be set to disable the `SafEth.unstake` function
L05 | ETH that is accidentally sent to the `SafEth` contract cannot be withdrawn
L06 | `SafEth.stake` function can cause the user to lose his staked ETH without receiving shares
NC01 | Bad variable naming in `Reth.sol`
NC02 | Unnecessary arithmetic operations inside the function `Reth.ethPerDerivative`
NC03 | Lack of event emission after critical `initialize` function
NC04 | Consider adding a timelock to the `SafEth.setMaxSlippage` function and other critical functions
NC05 | Inconsistent use of the types `uint` and `uint256`
NC06 | Open TODOs in unit tests for `SafEth.sol`
NC07 | ERC20 tokens accidentally sent to the contract can't be recovered
NC08 | Contracts that extend interfaces should override its methods
NC09 | Order of functions not complying with the Solidity Style Guide
NC10 | Named imports for modern code
NC11 | Grouped imports
NC12 | Use scientific notation rather than exponentiation

## Low

### L01: Bad english in comment in `Reth.sol`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L117

The comment on line 117 in the Reth contract is using bad english and incorrectly reads:

> Check whether or not rETH deposit pool has room users amount

It should read something like:

> Check whether or not the rETH deposit pool has room for the users amount

### L02: Missing the `@return` NatSpec tag for some functions in `Reth.sol`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L117-L120

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L153-L156

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L207-L211

The functions `Reth.poolCanDeposit`, `Reth.deposit` and `Reth.ethPerDerivative` are missing the `@return` NatSpec tag.

It is recommended to add the missing NatSpec tags, because Etherscan (etherscan.io) is using the information from the NatSpac tags, but if they are missing, it will not work for Etherscan.

### L03: Misleading and wrong `@notice` NatSpec tag for the function `SafEth.adjustWeight`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L158

The `@notice` tag on line 158 for the function `SafEth.adjustWeight` is wrong.

It should read:

```solidity
// SafEth
158        @notice - Adjusts the weight for a derivative
```


### L04: `maxSlippage` can be unfavorable for the user and can be set to disable the `SafEth.unstake` function

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202-L206

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48-L49

The owner of the `SafEth.sol` contract can set the slippage with the `SafEth.setMaxSlippage` function (SafEth.sol line 202) to any value. If the value for the slippage is set too high, it is unfavorable or nonsensical for the user.

For example the slippage can be set to 99% which would be unfavorable for the user.

Another problem is that if the `maxSlippage` is set too high, the unstake function gets disabled (DOS condition) due to an underflow revert that occurs. This happens, if the `maxSlippage` is set to higher than `10 ** 18`. When a user calls the `SafEth.unstake` function (SafEth.sol, line 108), the `withdraw` function for all derivatives is called (SafEth.sol, line 118). Then in `WstEth.withdraw` the underflow revert happens on line 60, if `maxSlippage` is set higher than `10 ** 18`. So the unstake function reverts due to this DOS condition and is disabled for the users.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L60


Recommended mitigation for `WstEth.sol` (the other derivatives contracts like `Reth.sol` etc should be changed in a similar way):

```solidity
// WstEth
48    function setMaxSlippage(uint256 _slippage) external onlyOwner {
+         require(maxSlippage < 10 ** 18, "Slippage too high");
+         require(maxSlippage <= maxAllowedValue); // maxAllowedValue that makes sense
49        maxSlippage = _slippage;
50    }

```

Yet another problem is that `minOut` can become 0, if `maxSlippage` is set to `10 ** 18`, because of a multiplication by 0 (WstEth.sol line 60). There the `stEthBal` would be multiplied with 0 if `maxSlippage` is `10 ** 18`:

```solidity
// minOut is 0, because stEthBal is multiplied with 0
uint256 minOut = (stEthBal * (10 ** 18 - 10 ** 18)) / 10 ** 18;
```

### L05: ETH that is accidentally sent to the `SafEth` contract cannot be withdrawn

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L246

If ETH is accidentally sent to the contract `SafEth` it cannot be withdrawn and is locked in the contract forever. 

Consider to only allow whitelisted addresses to send ETH to the `SafEth` contract.

Recommended mitigation:

```solidity
SafEth
+      mapping (address => bool) private receiveETHWhitelist;

182    function addDerivative(
183        address _contractAddress,
184        uint256 _weight
185    ) external onlyOwner {
186        derivatives[derivativeCount] = IDerivative(_contractAddress);
+          receiveETHWhitelist[_contractAddress] = true; // add contract to whitelist


247    receive() external payable {
+          require(receiveETHWhitelist[msg.sender], "Not allowed to send ETH");
```

This problem doesn't apply to the derivative contracts (`Reth`, `WstEth`, `SrfxEth`), where ETH can't be locked, because the contracts always send the whole ETH balance when their `withdraw` function is called.




### L06: `SafEth.stake` function can cause the user to lose his staked ETH without receiving shares

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L99

The `SafEth.stake` function doesn't check whether `mintAmount` is bigger than 0. Thus it's possible and there are cases, where the user stakes ETH, but receives 0 shares, since `mintAmount` is 0.

An example for this is, when each derivative that was added to `SafEth` contract, has a 0 weight. In this case, when iterating over the derivatives, `totalStakeValueEth` (line 83) would remain 0, because the 0 `weight` always triggers the continue statement (line 87):

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L83

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L87

Then the calculation for the `mintAmount` (line 98) would result in `mintAmount` to be 0, because `totalStakeValueEth` is 0:

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L98

```solidity
SafEth
+ // totalStakedValue is 0 below, resulting in mintAmount to be 0
98        uint256 mintAmount = (0 * 10 ** 18) / preDepositPrice;
```

Then the user would receive 0 shares for his staked ETH, because `mintAmount` is 0, and because the `_mint` function doesn't revert when minting 0 amount:

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L99

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/dd8ca8adc47624c5c5e2f4d412f5f421951dcc25/contracts/token/ERC20/ERC20Upgradeable.sol#L256-L269

Recommendation:

Add a require statement to the `SafeEth.stake` function that checks that `mintAmount` is bigger than 0.

```solidity
SafEth
98        uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
+         require(mintAmount > 0, "No shares to mint");
```


## Non-critical

### NC01: Bad variable naming in `Reth.sol`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L197-L199

There are bad variable names for the balance before and the balance after depositting. The variables are named `rethBalance1` and `rethBalance2`.

The variable names should be changed to something like:

```solidity
// Reth
197            uint256 rethBalanceBefore = rocketTokenRETH.balanceOf(address(this));
198            rocketDepositPool.deposit{value: msg.value}();
199            uint256 rethBalanceAfter = rocketTokenRETH.balanceOf(address(this));
```

### NC02: Unnecessary arithmetic operations inside the function `Reth.ethPerDerivative`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L215

Inside the function `Reth.ethPerDerivative` on line 215, the `poolPrice()` is multiplied with `10 ** 18` and then divided by ` 10 ** 18`. The multiplier and divisor is the same value and can be removed. So just the `poolPrice()` should be returned.

### NC03: Lack of event emission after critical `initialize` function

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L48-L56

To record the init parameters for offchain monitoring and transparency reasons, please consider emitting an event at the end of the `initialize` function.

### NC04: Consider adding a timelock to the `SafEth.setMaxSlippage` function and other critical functions

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202

Consider adding a timelock to the `SafEth.setMaxSlippage` function to give users time to react and adjust. By adding a timelock, the level of trust required is reduced and the risk for users is therefore decreased. This measure also shows to the users, that the project is legitimate.

### NC05: Inconsistent use of the types `uint` and `uint256`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L171-L173

On line 171 the type `uint` is used for the variable `rethPerEth`, but on line 173 the type `uint256` is used for the variable `minOut`.

It is recommended to use the type `uint256` for the variable `rethPerEth` in order to make it consistent.

### NC06: Open TODOs in unit tests for `SafEth.sol`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/test/SafEth.test.ts#L491

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/test/SafEth.test.ts#L516

The unit tests `SafEth.test.ts` for the `SafEth` contract contain unresolved TODOs.

It is recommended to remove the TODOs from code or fix them. Or add a link to a proper issue in your TODO.

### NC07: ERC20 tokens accidentally sent to the contract can't be recovered

Tokens that are accidentally sent to your contracts, which happened to many popular projects, can't be recovered.

Adding some code to your critical contracts, that enables recovering those ERC20 tokens, is recommended.

```solidity
// SafEth
/**
  * @notice - recovers ERC20 tokens trapped in this contract
  * @param receiver - the receiver of the trapped tokens
  * @param token - the token to recover
  * @return - success value for the operation.
  *
 */
function recoverTrappedERC20(
    address receiver,
    address token) public onlyOwner returns (bool) {
    IERC20(token).transfer(receiver, IERC20(token).balanceOf(address(this));
    return true;
}
```

### NC08: Contracts that extend interfaces should override its methods

The derivative contracts `Reth`, `WstEth`and `SfrxEth` are implementing the `IDerivative` interface methods. They should be using the override keyword to indicate which methods are implementing the interface.

### NC09: Order of functions not complying with the Solidity Style Guide

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L246

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L244


Functions should be grouped according to their visibility and ordered:

- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private

The contracts in scope have the `receive` function at the end instead of after the constructor. Furthermore inside `Reth.sol` functions are not at all ordered and are featuring a random order.

### NC10: Named imports for modern code

Consider using named imports for modern code and better code readability.

E.g. `import "../../interfaces/IDerivative.sol";` can be rewritten as `import {IDerivative} from "../../interfaces/IDerivative.sol";`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L4

### NC11: Grouped imports

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L4-L15

Instead of using random ordered imports, like for example in `Reth.sol`, consider grouping the imports together. First importing all OpenZeppelin contracts, then all interfaces. Thus improving code readability.

Example:


```solidity
// Reth
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "../../interfaces/IDerivative.sol";
import "../../interfaces/frax/IsFrxEth.sol";
import "../../interfaces/IWETH.sol";
import "../../interfaces/rocketpool/RocketStorageInterface.sol";
import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
import "../../interfaces/uniswap/ISwapRouter.sol";
import "../../interfaces/uniswap/IUniswapV3Factory.sol";
import "../../interfaces/uniswap/IUniswapV3Pool.sol";
```

### NC12: Use scientific notation rather than exponentiation

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L98

For better code readability, it is recommended to use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18).

Note: To find all instances, search for `10 **` in the codebase, in the *.sol files.