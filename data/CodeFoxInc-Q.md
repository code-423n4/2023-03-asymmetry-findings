# Low-risk and Non-critical issues

## ****Codebase Impressions & Summary****

The protocol is built on several other projects and the the enhanced way created by the team to stake Ether is quite interesting. 

The protocol’s contracts are well structured and quite simplified and the Natspec documentation is quite well done.  These efforts made by the team made the codebase easier to be understood by the auditors. The overall quality of the documentation is good but an independent documentation rather than only the `README` file is preferred if possible. 

However, there are some minor modifications can be done to improve the healthiness of the codebase. 

I proposed findings which focused on several issues, e.g. Re-entrancy attack possibility, Upgradeability pattern, solidity version, trusted owner issue, code style and some issues for following the best practice of solidity, etc. If they are to be addressed, the code quality are supposed to be improved.  

## L-01 Make sure the `unstake()` function won’t get Re-entrancy attacked

### Vulnerability Details

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L108-L129](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L108-L129)

The function `unstake()` send ether to the external accounts. It has a risk of being used as re-entrancy attack. Although the function follows the CEI rule, it is recommended that import the ReentrancyGuard contract and stay safe. 

### Impact

The `unstake` function sends Ether to external accounts and has a risk of being Re-entrancy attacked. 

### Recommendation

Use OpenZeppelin’s [Re-entrancy modifier library](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/security/ReentrancyGuardUpgradeable.sol) to prevent such attack from happening. 

In `SafEth.sol` do this: 

```diff
+ import "@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";

        contract SafEth is
            Initializable,
            ERC20Upgradeable,
            OwnableUpgradeable,
            SafEthStorage,
+           ReentrancyGuardUpgradeable
        {
-           function unstake(uint256 _safEthAmount) external {
+           function unstake(uint256 _safEthAmount) external nonReentrant {
                // ...
            }
        }
```

## L-02 The `addDerivative()`, `adjustWeight()` and `rebalanceToWeights()` functions can be used to steal funds

### **Vulnerability details**

There are quite a lot of `onlyOwner` functions and specifically speaking, the `adjustWeight`, `addDerivative` and `rebalanceToWeights` function can be used to drain funds. So there is a centralized risk. 

The owner can add a new malicious derivative by calling `addDerivative` and then adjust the weight to e.g. 0/0/0/100. After that if he calls `rebalanceToWeights` function then all the funds can be drained. 

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L165-L175](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L165-L175)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182-L195](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182-L195)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L138-L155](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L138-L155)

### Impact

So there is a centralized risk. Owner can always drain funds. Owner needs to be trusted. 

### Recommendation

Mitigate the risk by at least using a multi-sig wallet for the owner. Plus, the team can create a plan to renounce the ownership in the future if it is possible.  

## N-01 Use a struct array structure instead of `derivativeCount`

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEthStorage.sol#L18](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEthStorage.sol#L18)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L84](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L84)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L171](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L186](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L187](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L191](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140)

create a data structure like this:  

```solidity
 struct Derivative{
        IDerivative derivative;
        uint256 weight;
}
```

and put them into an array: 

```solidity
Derivative[] public derivatives;
```

then you can use derivatives.length to get the length of the array when looping through the array to do something.
You can see a similar example in [solidity-by-example](https://solidity-by-example.org/structs/). 

This can make the code clean and simple to understand. 

Modify every loop in the codebase as below: 

```diff
-     for (uint i = 0; i < derivativeCount; i++)  
+     uint256 derivativeLength = derivatives.length; // cache the value to save gas
+     for (uint i; i < derivativeLength; ++i)    
```

Get the value in the derivatives in this way: 

```solidity
	derivatives[i].derivative;
	derivatives[i].weight;
```

## N-02 Use `uint256` instead of `uint` following the best practice

I noticed in the code base, `uint` is in a lot of places. Please follow the best practice of solidity and change it to `uint256` in the files below.  

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol)

## N-03 Non-critical Issues Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

Change the `10**18` into `1e18`, and change other similar codes accordingly. 

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L44](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L44)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L171](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L171)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L173-L174](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L173-L174)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L54-L55](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L54-L55)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L80-L81](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L80-L81)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L94](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L94)

## N-04 all magic numbers can be set as constants.

I noticed in the code base, there are a lot of magic numbers, like `10 ** 18`,  `5 * 10 ** 17` and `200 * 10 ** 18`, etc. 

They can be done as a constant value instead to increase the code’s readability. 

```solidity
uint256 consttant PRICE_OF_ONE = 1e18; 
uint256 constant INITIAL_MAX_AMOUNT = 200 * 1e18;
uint256 constant INITIAL_MIN_AMOUNT = 5 * 1e17;
```

## N-05 Usage of a floating solidity version is not good and should be fixed

### **Context**

 All contracts

### **Description**

The project is compiled with different versions of solidity, which is not recommended due to undefined behaviors as a result of it. Should set it as a fixed version. 

Besides the below contracts, all the contracts are still using the relatively old floating version. 

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEthStorage.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEthStorage.sol#L2)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L2)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L2](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L2)

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; [https://github.com/ethereum/solidity/blob/develop/Changelog.md](https://github.com/ethereum/solidity/blob/develop/Changelog.md)

### **Recommendation**

Old version of Solidity is used , newer version can be used `(0.8.18)`. 

It is recommended use a static version of solidity instead of a floating version in production phase. 

As recommended in the [slither document](https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-versions-of-solidity), `0.8.18` is a good option. 

```diff
- pragma solidity ^0.8.13;
+ pragma solidity 0.8.18;
```

## N-06 Wrong document description

The description is not for this `adjustWeight` function and should be for the `addDerivative` function. 

I think the right description should be: `@notice - Adjust the weight of a derivative`. 

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L157-L158](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L157-L158)

## N-07 Using UUPS upgradeable pattern rather than Transparent Proxy pattern

I noticed in the `upgradeHelpers.ts` script file, Transparent Proxy pattern is used. 

I strongly recommend use UUPS upgradeable pattern instead of Transparent Proxy pattern. 

The reason is because UUPS is considered superior to the Transparent Proxy pattern because it offers lower gas costs, simplified storage, enhanced security, a reduced attack surface, and greater flexibility. 

[OpenZeppelin team also made some very good comments about this](https://docs.openzeppelin.com/contracts/4.x/api/proxy#transparent-vs-uups).

## N-08 Use constant value instead of hardcoded value

### Context

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L51](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L51)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L45](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L45)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L42](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L42)

### Description

Use constant value instead of hardcoded value. 

### Recommendation

Modify the `Reth`, `WstEth`, and `SfrxEth` contract as below. This is an example for `Reth` contract. 

```diff
// Reth.sol file
+   string public constant NAME = "RocketPool";

...

		function name() public pure returns (string memory) {
-        return "RocketPool";
+        return NAME; // @audit non-critical instead of hardcoding the name, we should use a constant variable.
    }
```

## N-09 lack of documentation

If there is a documentation besides the README file, it would be wonderful. 

## N-10 Should initialize the interface and save it into the storage

### Context

This is an example which was given in the [Rocket Pool](https://docs.rocketpool.net/developers/usage/contracts/contracts.html#interacting-with-rocket-pool) documentation. 

```solidity
import "RocketStorageInterface.sol";

contract Example {

    RocketStorageInterface rocketStorage = RocketStorageInterface(0);

    constructor(address _rocketStorageAddress) public {
        rocketStorage = RocketStorageInterface(_rocketStorageAddress);
    }
}
```

### Code Snippet

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L29-L45](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L29-L45)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L23-L39](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L23-L39)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L20-L36](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L20-L36)

### Recommendation

In the `Reth.sol` file, we should follow the practice in the official documentation. 

```diff
...

contract Reth is IDerivative, Initializable, OwnableUpgradeable {

		uint256 public maxSlippage;
+   RocketStorageInterface rocketStorage;

...

		/**
        @notice - Function to initialize values for the contracts
        @dev - This replaces the constructor for upgradeable contracts
        @param _owner - owner of the contract which handles stake/unstake
    */
    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
+       rocketStorage = RocketStorageInterface(ROCKET_STORAGE_ADDRESS);
    }

...
```

And in other part of the contract, you should also use the initialized `rocketStorage` instead. 

In SfrxEth.sol file and WstEth.sol file, the same change should be make. 

```diff
...

		uint256 public maxSlippage;
+		IERC20 SfrxEth;

...		

		/**
        @notice - Function to initialize values for the contracts
        @dev - This replaces the constructor for upgradeable contracts
        @param _owner - owner of the contract which handles stake/unstake
    */
    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
+				SfrxEth = IERC20(SFRX_ETH_ADDRESS);
    }
```

```diff
...

		uint256 public maxSlippage;
+		IWStETH wstETH;
+		IERC20 stEthToken;

...		

		/**
        @notice - Function to initialize values for the contracts
        @dev - This replaces the constructor for upgradeable contracts
        @param _owner - owner of the contract which handles stake/unstake
    */
    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
+				wstETH = IWStETH(WST_ETH);
+       stEthToken = IERC20(STETH_TOKEN)
    }
```

## N-11 Usage of interface is not right

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L94](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L94)

Although it is alright to use `IERC20` here, but it is recommended that should keep the interface consistent. 

```diff
		/**
        @notice - Total derivative balance
     */
    function balance() public view returns (uint256) {
-        return IERC20(WST_ETH).balanceOf(address(this));
+        return IWStETH(WST_ETH).balanceOf(address(this));
    }
```
