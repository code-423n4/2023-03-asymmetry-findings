## Findings summary
Name | Finding | Instances | Gas saved
--- | --- | --- | ---
[G-1] | Duplicated require()/revert()/assert() checks should be refactored to a modifier or an internal function | 3 | 60282
[G-2] | Use custom errors instead of revert strings | 10 | 2540
[G-3] | Setting the constructor to payable | 4 | 864
[G-4] | Avoid using public for immutable/constant state variables | 11 | 272899
[G-5] | Use calldata instead of memory for external function parameters | 2 | /
[G-6] | Functions guaranteed to revert for normal users should be marked as `payable` | 17 | 408
[G-7] | use ternary operators rather than `if/else` | 3 | 39
[G-8] | No need to initialize variables to their default values | 7 | /
[G-9] | Cache rocket-style string encoded into immutables | 6 | 864

## Findings details
### [G-1] Duplicated require()/revert()/assert() checks should be refactored to a modifier or an internal function


Duplicated require, revert, or assert messages should be refactored to an internal or private function in order to save some gas on deployment. The more duplicated it is, the greater the savings, but this always saves gas starting from two duplication.

`contracts/SafEth/derivatives/Reth.sol`
200:12

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L200

```solidity
              uint256 rethBalance2 = rocketTokenRETH.balanceOf(address(this));
>             require(rethBalance2 > rethBalance1, "No rETH was minted");
              uint256 rethMinted = rethBalance2 - rethBalance1;
```

### Gas saved (per instance on deployment)

20094

<details>
<summary>Locations</summary>
<br>

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L66

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L77

</details>

### [G-2] Use custom errors instead of revert strings


Solidity 0.8.4 added the custom errors functionality, which can be used instead of revert strings, resulting in big gas savings on errors since they all just consist of the 4 bytes signature of the error, similar to functions. They can then be decoded thanks to the ABI.

`contracts/SafEth/SafEth.sol`
64:8

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L64

```solidity
      function stake() external payable {
>         require(pauseStaking == false, "staking is paused");
          require(msg.value >= minAmount, "amount too low");
```

### Gas saved (per instance)

254

<details>
<summary>Locations</summary>
<br>

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L200

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L87

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L113

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L77

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L66

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L66

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L109

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L65

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L127

</details>

### [G-3] Setting the constructor to payable


Marking the constructor as payable removes any value check from the compiler and saves some gas on deployment.

`contracts/SafEth/derivatives/WstEth.sol`
24:4

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L24

```solidity
      /// @custom:oz-upgrades-unsafe-allow constructor
>     constructor() {
>         _disableInitializers();
>     }
  
```

### Gas saved (per instance)

216

<details>
<summary>Locations</summary>
<br>

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L38

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L27

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L33

</details>

### [G-4] Avoid using public for immutable/constant state variables


Public state variable are generating a getter function which costs more gas on deployment. Some variables may not need to require a getter function.

`contracts/SafEth/derivatives/SfrxEth.sol`
18:4

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L18

```solidity
          0x5E8422345238F34275888049021821E8E08CAa1f;
>     address public constant FRX_ETH_CRV_POOL_ADDRESS =
>         0xa1F8A6807c402E4A15ef4EBa36528A3FED24E577;
      address public constant FRX_ETH_MINTER_ADDRESS =
```

### Gas saved (per instance on deployment)

24809

<details>
<summary>Locations</summary>
<br>

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L26

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L16

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L15

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L20

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L20

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L24

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L17

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L22

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L13

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L14

</details>

### [G-5] Use calldata instead of memory for external function parameters


By the use of the `memory` keyword, all of the variables from the function parameter are copied to the memory by using the opcode `CALLDATACOPY`. This opcode gas cost grows linearly as a function of the number of slots to copy plus the memory expansion cost which can grow quadratically if there are a lot of slots to copy. If there is no need to alter the variables and store them somewhere, then we can safely load them from the calldata. See on evm.codes for more informations: https://www.evm.codes/#37

`contracts/SafEth/SafEth.sol`
50:8

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L50

```solidity
          string memory _tokenName,
>         string memory _tokenSymbol
      ) external initializer {
```

<details>
<summary>Locations</summary>
<br>

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L49

</details>

### [G-6] Functions guaranteed to revert for normal users should be marked as `payable`


When a function is restricted to only one account, it's less expensive to mark it as `payable` since it's going to remove any opcode relative to the msg.value check

`contracts/SafEth/derivatives/SfrxEth.sol`
60:48

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L60

```solidity
       */
>     function withdraw(uint256 _amount) external onlyOwner {
          IsFrxEth(SFRX_ETH_ADDRESS).redeem(
```

#### Gas saved (per instance)

24

<details>
<summary>Locations</summary>
<br>

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L214

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L232

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L185

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L138

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L48

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L107

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L205

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L58

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L56

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/WstEth.sol#L73

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L223

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L51

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L241

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L168

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/SfrxEth.sol#L94

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L156

</details>

### [G-7] use ternary operators rather than `if/else`


The `if/else` statement runtime gas cost is higher than a ternary operator in some cases.

`contracts/SafEth/derivatives/Reth.sol`
212:8

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L212

```solidity
      function ethPerDerivative(uint256 _amount) public view returns (uint256) {
>         if (poolCanDeposit(_amount))
>             return
>                 RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
>         else return (poolPrice() * 10 ** 18) / (10 ** 18);
      }
```

### Gas saved (per instance)

13

<details>
<summary>Locations</summary>
<br>

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L79

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/derivatives/Reth.sol#L170

</details>

### [G-8] No need to initialize variables to their default values


In the EVM, everything is an empty word (0) of 32 bytes. Variables in Solidity does not need to be assigned explicitely to their default value as they already have these values and incurs a higher gas overhead without the optimizer turned on.

`contracts/SafEth/SafEth.sol`
71:13

https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L71

```solidity
          // Getting underlying value in terms of ETH for each derivative
>         for (uint i = 0; i < derivativeCount; i++)
              underlyingValue +=
```

<details>
<summary>Locations</summary>
<br>

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L113

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L84

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L140

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L191

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L147

- https://github.com/code-423n4/2023-03-asymmetry/tree/main/contracts/SafEth/SafEth.sol#L171

</details>

### [G-9] Cache rocket-style string encoded into immutables

RocketPool introduced the use of keccak-ed concatenated strings for their eternal storage.
This ensure not to have any conflict with any other contract because those are unique.
They also also allow to set any key to any value and their library provide helpers to cast the variables to the correct types. But this can be optimized by writing it as an immutable, since Solidity supports it and will be evaluated at compile-time.

`contracts/SafEth/derivatives/Reth.sol`
70:25

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L70

```solidity
            RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
>               keccak256(
>                   abi.encodePacked("contract.address", "rocketTokenRETH")
>               )
            );
```

### Gas saved (per instance)

144

<details>
<summary>Locations</summary>
<br>

- 
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L124-L126
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L135-L140
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L161-L163
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L190-L192
- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L232-L234

</details>