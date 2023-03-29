Q/A 1) A low liquidity Pool is used for rETH.
 The pool only has 5 Millions of Liquidity https://coinmarketcap.com/dexscan/ethereum/0xa4e0faa58465a2d369aa21b3e42d43374c6f9613/
Whereas the same pool on Balancer has 80 millions of liquidity
https://app.balancer.fi/#/ethereum/pool/0x1e19cf2d73a72ef1332c882f20534b6519be0276000200000000000000000112
Use the balancer pool instead. To have better liquity and not run into a DOS caused by the slippage.

Q/A 2 ) Wrong logic for rETH price
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L211
For ethPerDerivative(uint256 _amount) in rETH, it makes no sense to check if the amount can be deposited in the Rocker Pool Deposit Pool. What matter is if the amount can be withdrawn from the pool. For instance right now there is 5000 eth in the deposit pool that can be used with no slippage to convert rETH to ETH. But since the deposit pool is full, the current logic prevents using it. And getting the price from it .

Q/A 3 ) In the stake function, the price of derivatives deposited that matter is not the one used to deposit, but the one used if we want to sell them. So generally a chainlink Oracle would be better.

Q/A 4 ) In case of Depeg, buy the derivatives on the market to accumulate more ETH instead of minting new derivatives. 

