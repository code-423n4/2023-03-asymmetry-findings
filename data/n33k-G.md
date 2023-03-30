## Use constant variables for getAddress keys in Reth.sol

Take this one for exmaple.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L68-L72
```
            RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
                keccak256(
                    abi.encodePacked("contract.address", "rocketTokenRETH")
                )
            );
```

Optimize steps:
Define a constant variable and use the variable instead of calculate the key everytime.

```
bytes32 public constant KEY_rocketTokenRETH=keccak256(abi.encodePacked("contract.address", "rocketTokenRETH"));

RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(KEY_rocketTokenRETH);
```

Other plances that can be optimzed:

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L124-L126

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L135-L138

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L161-L163

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L190-L192

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L232-L234