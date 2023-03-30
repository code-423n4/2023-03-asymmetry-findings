# Balance variable instead of repeated calls.
[SafEth.sol: L71-L75](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71-L75)

```
for (uint i = 0; i < derivativeCount; i++) { 
    uint256 balance = derivatives[i].balance();
    underlyingValue += (derivatives[i].ethPerDerivative(balance) * balance) / 10 ** 18;
    }
```

Before:
```
|  SafEth    ·  stake               ·     437702  ·     651569  ·        524700  ·          348  ·      
```
After:
```
|  SafEth    ·  stake               ·     429365  ·     643415  ·        516485  ·          348  ·          
``` 

# Only instantiate Rocketpool variables if conditions are met.
[Reth.sol: L156-L204](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L156-L204)

```
function deposit() external payable onlyOwner returns (uint256) {
            if (!poolCanDeposit(msg.value)) {
            uint rethPerEth = (10 ** 36) / poolPrice();

            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
                ((10 ** 18 - maxSlippage))) / 10 ** 18);

            IWETH(W_ETH_ADDRESS).deposit{value: msg.value}();
            uint256 amountSwapped = swapExactInputSingleHop(
                W_ETH_ADDRESS,
                rethAddress(),
                500,
                msg.value,
                minOut
            );

            return amountSwapped;
        } else {
            // Per RocketPool Docs query addresses each time it is used
            address rocketDepositPoolAddress = RocketStorageInterface(
            ROCKET_STORAGE_ADDRESS
            ).getAddress(
                keccak256(
                    abi.encodePacked("contract.address", "rocketDepositPool")
                )
            );

            RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(
                rocketDepositPoolAddress
            );
            address rocketTokenRETHAddress = RocketStorageInterface(
                ROCKET_STORAGE_ADDRESS
            ).getAddress(
                    keccak256(
                        abi.encodePacked("contract.address", "rocketTokenRETH")
                    )
                );
            RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(
                rocketTokenRETHAddress
            );
            uint256 rethBalance1 = rocketTokenRETH.balanceOf(address(this));
            rocketDepositPool.deposit{value: msg.value}();
            uint256 rethBalance2 = rocketTokenRETH.balanceOf(address(this));
            require(rethBalance2 > rethBalance1, "No rETH was minted");
            uint256 rethMinted = rethBalance2 - rethBalance1;
            return (rethMinted);
        }
    }
```

Before:
```
|  SafEth    ·  stake               ·     437702  ·     651569  ·        524700  ·          348  ·      
```

After:
```
|  SafEth    ·  stake               ·     437702  ·     650509  ·        523993  ·          348  ·
```

