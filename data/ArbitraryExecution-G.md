### Redundant external `balanceOf` call

Gas savings in the [`deposit`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L94) function in `SfrxEth.sol` can be had by returning the return value from the [`submitAndDeposit`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L101) external call. This would eliminate two external `balanceOf` external calls, as well as the [subtraction operation](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L105).

### Redundant external `balanceOf` call

Gas savings in the [`withdraw`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56) function in `WstEth.sol` can be had by saving off the return value of the [`unwrap`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L57) call and using that return value instead of making an additional [`balanceOf`](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L58) external call to the `STETH_TOKEN` address.

### Use an Array instead of a mapping

The following loop could be made more gas efficient by changing the `derivatives` variable type:

```
    for (uint i = 0; i < derivativeCount; i++)
        underlyingValue +=
            (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                derivatives[i].balance()) /
            10 ** 18;
```

A storage pointer can't be used here because its a mapping so the elements won't be contiguous. 
If the `derivatives` variable is changed from a mapping to an array a storage pointer could be used and would result in significant gas savings.

