# Low Risk and Non-Critical Issues

## N-01 Better to use `__ownable_init()` instead of `_transferOwnership()` as this is the recommended way to initialize the Ownable contract.
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L53

##### SafEth::initialize

```solidity
File: contracts\SafeEth.sol
48:     function initialize(
49:         string memory _tokenName,
50:         string memory _tokenSymbol
51:     ) external initializer {
52:         ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
53:         _transferOwnership(msg.sender);


```


## N-02 No event triggered when `minAmount` and `maxAmount` are set
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L54-L55

##### SafeEth::initialize
```solidity
File: contracts\SafeEth.sol
48:     function initialize(
49:         string memory _tokenName,
50:         string memory _tokenSymbol
51:     ) external initializer {
52:         ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
53:         _transferOwnership(msg.sender);
54:         minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
55:         maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
56:     }

```
