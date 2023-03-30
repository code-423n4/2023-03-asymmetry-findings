# Gas Optimizations

## Summary

|               | Issue         | Instances     |
| :-------------: |:-------------|:-------------:|
| 1  | Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct | 2 |
| 2  | `storage` variable should be cached into `memory` variables instead of re-reading them  |  4 |
| 3  | Use `unchecked` blocks to save gas  | 1 |
| 4  | `derivatives[i].balance()` should be cached into memory  | 2 |
| 5  | `derivativeCount` should not be read at each loop iteration | 7 |
| 6  | `memory` values should be emitted in events instead of `storage` ones  | 4 |
| 7  | `public` functions not called by the contract should be declared `external` instead | 8 |


## Findings

### 1- Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct :

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

There are 2 instances of this issue :

File: SafEth.sol [Line 22-23](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEthStorage.sol#L22-L23)
```
mapping(uint256 => IDerivative) public derivatives; // derivatives in the system
mapping(uint256 => uint256) public weights; // weights for each derivative
```

These mappings could be refactored into the following struct and mapping for example :

```
struct Derivative {
    uint256 weight;
    IDerivative derivative;
}
    
mapping(uint256 => Derivative) public derivatives;
```


### 2- `storage` variable should be cached into `memory` variables instead of re-reading them :

The instances below point to the second+ access of a state variable within a function, the caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read, thus saves **100gas** for each instance.

There are 4 instances of this issue :

File: SafEth.sol [Line 71-84](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71-L84)

In the code linked above the value of `derivativeCount` is read multiple times (2) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

File: SafEth.sol [Line 140-147](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140-L147)

In the code linked above the value of `derivativeCount` is read multiple times (2) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

File: SafEth.sol [Line 186-187](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L186-L187)

In the code linked above the value of `derivativeCount` is read multiple times (2) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

File: SafEth.sol [Line 191-194](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191-L194)

In the code linked above the value of `derivativeCount` is read multiple times (2) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

### 3- Use `unchecked` blocks to save gas :

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

There is 1 instance of this issue:

File: SafEth.sol [Line 188](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L188)
```
derivativeCount++;
```

The increment of `derivativeCount` cannot realisticaly overflow as they represent the numbers of derivative contracts, so the operation should be marked as `unchecked` to save gas. 


### 4- `derivatives[i].balance()` should be cached into memory :

`derivatives[i].balance()` is a call to an external contract which cost a lot of gas, so when the actual derivative contract balance is not changing `derivatives[i].balance()` should only be called once and its value should be cached into a memory variable to save gas instead of making this call multiple times.

There are 2 instances of this issue :

File: SafEth.sol [Line 73-74](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L73-L74)
```
underlyingValue +=
    (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
        derivatives[i].balance()) /
    10 ** 18;
```

File: SafEth.sol [Line 141-142](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L141-L142)
```
if (derivatives[i].balance() > 0)
    derivatives[i].withdraw(derivatives[i].balance());
```


### 5- `derivativeCount` should not be read at each loop iteration :

The value of `derivativeCount` is read directly from storage at each iteration of the for loops which will cost a lot of gas, its value should be cached into memory variable instead.

There are 7 instances of this issue :

```
File: SafEth.sol

71      for (uint i = 0; i < derivativeCount; i++)
84      for (uint i = 0; i < derivativeCount; i++)
113     for (uint256 i = 0; i < derivativeCount; i++)
140     for (uint i = 0; i < derivativeCount; i++)
147     for (uint i = 0; i < derivativeCount; i++)
171     for (uint256 i = 0; i < derivativeCount; i++)
191     for (uint256 i = 0; i < derivativeCount; i++)
```


### 6- `memory` values should be emitted in events instead of `storage` ones :

The values emitted in events shouldn’t be read from storage but the existing memory values should be used instead, this will save **~100 GAS**.

There are 4 instances of this issue :

File: SafEth.sol [Line 216](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L216)
```
emit ChangeMinAmount(minAmount);
```

File: SafEth.sol [Line 225](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L225)
```
emit ChangeMaxAmount(maxAmount);
```

File: SafEth.sol [Line 234](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L234)
```
emit StakingPaused(pauseStaking);
```

File: SafEth.sol [Line 243](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L243)
```
emit UnstakingPaused(pauseUnstaking);
```

### 7- `public` functions not called by the contract should be declared `external` instead :

The `public` functions that are not called inside the contract should be declared `external` instead to save gas.

There are 8 instances of this issue:

```
File: Reth.sol

50     function name() public pure returns (string memory)
211    function ethPerDerivative(uint256 _amount) public view returns (uint256)
221    function balance() public view returns (uint256)

File: SfrxEth.sol

44     function name() public pure returns (string memory)
122    function balance() public view returns (uint256)

File: WstEth.sol

41     function name() public pure returns (string memory)
86     function ethPerDerivative(uint256 _amount) public view returns (uint256)
93     function balance() public view returns (uint256)
```