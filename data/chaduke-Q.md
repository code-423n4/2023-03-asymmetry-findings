QA1. The protocol lacks the method ``removeDerivative()``. Setting the weight for a derivative contract to ZERO will not do it since a user can still unstake from a derivative contract with zero weight.

Mitigation: 1) Call ``rebalanceToWeights()`` after any weight of the derivative contract is adjusted. 
In this way, zero-weight derivative contract will have zero asset in it; or 2) explicitly implement a new method ``removeDerivative()``. The later is preferred. 


QA2. The protocol assumes that each derivative has 18 decimals, so the protocol will break when the underlying derivative has different decimals. 

For example, the following code for ``stake()`` assumes a 18 decimals for the underlying derivative:
```javascript
 uint256 underlyingValue = 0;

        // Getting underlying value in terms of ETH for each derivative
        for (uint i = 0; i < derivativeCount; i++)
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                    derivatives[i].balance()) /
                10 ** 18;
```

Mitigation: add a function decimals() for each derivative contract and use it to calculate different quantities such as ``underlyingValue``.

QA3. The ``stake()`` function has a precision loss issue due to the use of divide-before-multiplication.

(https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L63-L101](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L63-L101)

First, it calculates ``preDepositPrice`` as L81:
```javascript
preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
```

Then, it calculates the number of shares of ``safETH`` as:
```javascript
 uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```

Mitigation: 
To avoid larger precision loss and twice divisions, the correct formula should be
```javascript
 uint256 mintAmount;
if(totalSupply == 0) minAmount = (totalStakeValueEth;
else minAmount =  totalStakeValueEth * totalSupply / underlyingValue;
```

QA4: If a user sent ETH to the ``WstEth`` by mistake, it will be withdrawn by the user who unstakes from ``WstEth`` at L63 of the following withdraw() function:

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56-L67](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56-L67)

Mitigation: 
Delete the  ``receive()`` function from ``WstEth`` so that no user can send ETH to the contract directly.
For the same reason, ``receive()`` should be deleted from ``SfrxEth`` as well.

QA5. swapExactInputSingleHop() might not work for some tokens such as 
USDT and KNC, which do not allow approving an amount M > 0 when an existing amount N > 0 is already approved.

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L83-L102](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L83-L102)

Mitigation: approve to zero first and then approve to the needed amount:
```diff
function swapExactInputSingleHop(
        address _tokenIn,
        address _tokenOut,
        uint24 _poolFee,
        uint256 _amountIn,
        uint256 _minOut
    ) private returns (uint256 amountOut) {
+        IERC20(_tokenIn).approve(UNISWAP_ROUTER, 0);
        IERC20(_tokenIn).approve(UNISWAP_ROUTER, _amountIn);
        ISwapRouter.ExactInputSingleParams memory params = ISwapRouter
            .ExactInputSingleParams({
                tokenIn: _tokenIn,
                tokenOut: _tokenOut,
                fee: _poolFee,
                recipient: address(this),
                amountIn: _amountIn,
                amountOutMinimum: _minOut,
                sqrtPriceLimitX96: 0
            });
        amountOut = ISwapRouter(UNISWAP_ROUTER).exactInputSingle(params);
    }
```

QA6. Attackers might steal funds from ``SafEth`` by manipulating the price of ``Reth``.

The ``Reth`` price is based on ``ethPerDerivative()``:

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L211-L223](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L211-L223)

which is based on the following ``poolPrice()`` function:

``javascript
function poolPrice() private view returns (uint256) {
        address rocketTokenRETHAddress = RocketStorageInterface(
            ROCKET_STORAGE_ADDRESS
        ).getAddress(
                keccak256(
                    abi.encodePacked("contract.address", "rocketTokenRETH")
                )
            );
        IUniswapV3Factory factory = IUniswapV3Factory(UNI_V3_FACTORY);
        IUniswapV3Pool pool = IUniswapV3Pool(
            factory.getPool(rocketTokenRETHAddress, W_ETH_ADDRESS, 500)
        );
        (uint160 sqrtPriceX96, , , , , , ) = pool.slot0();
        return (sqrtPriceX96 * (uint(sqrtPriceX96)) * (1e18)) >> (96 * 2);
    }
```

However, an attacker can manipulate the spot price of ``sqrtPriceX96`` and trade to steal funds from the contract. It is always better to use TWAP price rather than the spot price. 

Mitigatin: use TWAP price rather than the spot price of ``reth``. 


QA7. addDerivative() and adjustWeight() might make different derivatives out of balance.

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L165-L175](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L165-L175)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182-L195](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182-L195)

More seriously, if some weight becomes ZERO, then that derivatives will only be withdrawn and not be deposited. 

Mitigation: 
Whenever some weight is changed, either by addDerivative() or by adjustWeight(), call ``rebalanceToWeights()`` to rebalance all the derivatives according to the new weight distribution.

QA8: There is no _gap[50] state variable for the upgradable contracts. 

It is important to declare a uint _gap[50] state variable for the following upgradable implementation contracts so that when they are upgraded with the introduction of new state variables, other inheriting contracts will not be disturbed. A storage gap allows new variables to be added in future versions of the contracts without changing the inheritance chain. see https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps for further explanation.

QA9: A compromised/malicious owner of SafEth can easily drain the whole safEth assets.
1) He can call ``addDrivative()`` to add a new malicious derivative contract ``BadContract``.

2) He calls adjustWeight() to set the weights of all other derivative contracts to zero and set the weight of ``BadContract`` to 100.

3) He calls rebalanceToWeights() to pull all the assets into ``BadContract``. 

4) The bad contracts send all the ETH to the attacker's private wallet.

Mitigation: 
``addDrivative()`` should only be allowed during deployment and the introduction of new derivatives should use proxy patterns to upgrade existing contracts. The practice should follow a standard proxy pattern best practice. 

QA10. setMaxSlippage() might have a silent failure. 

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202-L208](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202-L208)

The reason that setMaxSlippage() might have a silent failure is because ``derivatives`` state mapping variable, so even when ``derivativeIndex >= derivativeCount``, derivative[_derivativeIndex] will be equal to zero address. Calling a method on a zero address will have a silent success even thought nothing is accomplished. The event will also be emitted even though no slippage is actually set. 

Mitigation: check to make sure ``derivativeIndex < derivativeCount``.

QA11. Stake() and unstake() for the strategy contract will be blocked when one of its underlying derivative contract services become temporarily unavailable. 

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L63-L129](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L63-L129)


1. The stake() and unstake() function for the strategy contract will revert whenever one of the deposit() or withdraw() of the underlying derivative contract fails. As a result, stake() and unstake() become unavailable even though all other derivative contracts are still available. 

2. Set a weight to zero for that temporarily unavailable derivative contract is not the solution since the asset is still in that underlying contract and there is no way to rebalance it. Ignoring it won't work either since that means a user will unstake less amount of ETH - loss of funds

3. The design of the architecture should have the fault-tolerance property such that a user can still stake/unstake using the remaining derivative contracts without losing funds in ETH value.

Mitigation: a user should be able to stake/unstake using the remaining derivative contracts when one of them becomes unavailable  and resume to use it when it becomes available again. 

QA12. The amount of deposit() will impact the price of ``rEth``. As a result, a user might instead call deposit() with many small amounts for deposit to take advantage of the system with a favorable price to him. On the other hand, another user who call deposit() with a large amount will end up a worse price.
In summary, the system might be not fair to different users. 

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L211-L216](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L211-L216)

Consider the following scenario: 
1) Suppose rocketDAOProtocolSettingsDeposit.getMaximumDepositPoolSize()  is large enough and this maximum won't be exceeded for the next few months. 

2) Suppose rocketDAOProtocolSettingsDeposit.getMinimumDeposit() = 0.01e18

3) If a user needs to deposit 10e18, without splitting this amount, let's say the price would be 
1.1e18 (RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18)). 

4) On the other hand, if the user split 10e18 into 1001 subtransactions, as a result, each deposit piece is small than rocketDAOProtocolSettingsDeposit.getMinimumDeposit() = 0.01e18. Therefore, ``poolCanDeposit(_amount) = false``, and another price poolPrice() will be used instead, which could be more favorable, say 1.09e18. Using the later price, the user will get more ``rETH`` shares since 1.09e18 < 1.1e18.

5) In summary, depending on if one splits the whole deposit into small transactions or not, there might be two different prices of ``rETH`` used. This is not a fair system.

Mitigation: use only one source for ``rETH`` price so that the price is consistent regardless of the deposit amount. 
