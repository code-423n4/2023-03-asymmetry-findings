
1.
Avoid loops whenever possible, especially nested loops. Instead, use data structures that allow for constant-time lookup or caching. For example, in the `stake()` function, you could cache the `ethPerDerivative` value for each derivative to avoid the nested loop that calculates it.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L73

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L92


2. Use `if` statements instead of `require` statements for input validation. `require` statements are more expensive than `if` statements because they revert the transaction and refund the remaining gas. For example, in the `stake()` function, you could replace `require(pauseStaking == false, "staking is paused");` with if `(pauseStaking) revert("staking is paused")`.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L64

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L109

3. Use `transfer()` instead of `call()` when sending ether to other contracts. `transfer()` is safer and cheaper than `call()`. For example, in the `stake()` function, you could replace `derivative.deposit{value: ethAmount}()`; with `derivative.deposit{value: ethAmount}().transfer(ethAmount)`. 

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L91

4. Repeated code in events: The events `ChangeMinAmount()` and `ChangeMaxAmount()` both emit the same `uint256` value, which is the new `minAmount` or `maxAmount`. Instead of emitting the value in both events, you could create a single event, such as `ChangeAmount(uint256 amount, string message)`, where `message` is a string indicating whether it is the minimum or maximum amount being changed.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L21-L22

5. Use of continue: The continue statement in the `stake()` and `unstake()` function can be removed by using a more efficient way of iterating over the derivatives array, such as a mapping or an array of indexes.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L87

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L117

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L148

6. Use of payable functions: The `stake() and `unstake()` functions are marked as `payable`, which means they can receive Ether along with the function call. However, in the current implementation, the Ether received is immediately processed, so it may not be necessary to mark these functions as `payable`. You could consider removing the `payable` keyword if the Ether received is not needed immediately.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L63

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L108

7. In the `rethAddress()` function, you could store the `RocketStorageInterface` instance as a contract-level variable instead of creating a new instance each time the function is called. This would save gas because it would reduce the number of calls to the `getAddress()` function.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L68

8. In the `swapExactInputSingleHop()` function, you could add a check to see if the `_minOut` parameter is greater than zero. If it is not, you could skip the call to the Uniswap router, saving gas.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L83-L102

9. In the withdraw() function, you could remove the sent variable and use the require() statement to check if the transfer was successful. This would save gas because it would reduce the number of variables.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L110

10. In the `poolCanDeposit()` function, you could store the `rocketDepositPool` and `rocketProtocolSettings`` instances as contract-level variables instead of creating new instances each time the function is called. This would save gas because it would reduce the number of calls to the getAddress() function.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L120-L150

11. The `poolCanDeposit` function could be made public if it is intended to be called by other contracts. Otherwise, it could be made internal to save gas costs for not creating a new public function entry in the contract ABI.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L120

12. The `swapExactInputSingleHop` function could use `IERC20(_tokenIn).transferFrom(msg.sender, address(this), _amountIn)` instead of `IERC20(_tokenIn).approve(UNISWAP_ROUTER, _amountIn)` to save gas costs for approving the token transfer.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L90

13. The `withdraw()` function could be optimized by avoiding the use of the `approve` function to approve the `FRX_ETH_CRV_POOL_ADDRESS` to spend the `FRX_ETH_ADDRESS` tokens. Instead, the contract could hold the `FRX_ETH_ADDRESS` tokens directly and call the `exchange` function on the `IFrxEthEthPool` contract with the `FRX_ETH_ADDRESS` token allowance set to zero. This would save gas on the additional `approve` call and the subsequent `safeTransferFrom` call by the pool contract to transfer the tokens.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L60-L88

14. The `deposit()` function could be optimized by combining the two `IERC20` balance checks into a single call to `IERC20.balanceOf` by passing the contract address as an array of size 1 instead of calling it twice with the same argument.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L98-L100

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L102-L104

15. Consider adding `view` to the `setMaxSlippage()` function. If a function does not modify state, it is more gas efficient to declare it as `view` instead of `external`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51

16. The `withdraw()` function uses `call()` to send ETH to the owner of the contract. However, `transfer()` is a safer and cheaper option for sending ETH as it limits the amount of gas that can be consumed by the recipient contract.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L63

17. Consider using the `revert()` function instead of `require()`: In the `withdraw()` function, you can use the `revert()` function instead of `require()` as it is slightly cheaper in terms of gas usage.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L87
