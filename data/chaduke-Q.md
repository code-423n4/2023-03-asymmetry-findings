QA1. Once a derivative contract is added, there is no way to remove it. 

Mitigation: add a new function so that the owner can remove an existing derivative contract.

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