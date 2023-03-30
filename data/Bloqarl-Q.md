## QA Report

# [L-01] Use Ownable2StepUpgradeable instead of OwnableUpgradeable contract
In four contracts the `transferOwnership` is used in the `initialize()` function in order to use to change Ownership. It is, however, imported from `OwnableUpgradeable`. There is another Openzeppelin Ownable contract (Ownable2StepUpgradeable.sol) that also has `transferOwnership()` function.

There are four instances in contracts `SafEth.sol`, `Reth.sol`, `SfrxEth.sol` and `WstEth.sol`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L18
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L19
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L13
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L12


# [L-02] Use `_safeTransferOwnership` instead of `_transferOwnership`
Use `_safeTransferOwnership` function instead of `_transferOwnership`. It is more secure due to 2-stage ownership transfer.

There are four instances in contracts `SafEth.sol`, `Reth.sol`, `SfrxEth.sol` and `WstEth.sol`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L34
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L37
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L43
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L53

# [L-03] Avoid magic numbers
In order to make the code more readable for future changes, reviews or other usages, it is always convenient to not use numbers which are not named. That's why it's always define such numbers as a constant or variable depending of its usage.

For instance if you see in the code `minAmount = 5 * 10 ** 17;` and you didn't write it yourself it is very possible that the reviewer will not know what those numbers represent.

There are seven instances

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L54
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L55
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L180
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L238
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L241
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L38
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L75


# [L-04] Avoid shadowing inherited state variables
In `SafEth.sol`, there is a local variable named `totalSupply`, but there is a function named `_totalSupply()` in the inherited class with the same name. This use causes compilers to issue warnings, negatively affecting checking and code readability.

There is one instance in `SafEth.sol`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L77


# [L-05] There should be a check to validate totalSupply() is not zero

Division by zero is an exception that will result in a reverted transaction. Since it might cause issues, it is convenient to validate it before using it.

For instance we have `else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;`

In this case we should check first `totalSupply`

There are three instances in `SafEth.sol`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L81
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L88
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L116


# [L-06] Avoid address(0)
It is convenient to check the address received in the function's parameter to make sure it is not an `address(0)`.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L183


# [L-07] Hardcoded address causes no future updates in case new updates in their version
In case the addresses change due to reasons such as updating their versions in the future, addresses coded as constants cannot be updated, so it is recommended to add the update option with the onlyOwner modifier.

```
address public constant UNISWAP_ROUTER =
        0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45;
address public constant UNI_V3_FACTORY =
        0x1F98431c8aD98523631AE4a59f267346ea31F984;
```

# [L-08] Function's parameter is never used inside the function
There are two instances where we are passing a uint256 to a function, but inside that function the parameter is never used. It might potentially cause an unexpected behaviour of the code as it might be resulting in a misleading result.

For instance in this operation
```
 uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
            (10 ** 18 - maxSlippage)) / 10 ** 18;
```
We are passing `_amount` to `ethPerDerivative` but inside it's never used.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L74
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L111
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L86


# [L-09] Consider creating a constant for 10 \*\* 18 as it's repeated a lot
It is a good practice to extract as a constant or variable a piece of code that is repeated often. In this case `10**18` is constantly used. 

In case of having to change that to a different number, by having it as a constant it would only need to be changed in a place instead of in many, hence, reducing the potential human mistake.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L60
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L87
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L74
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L113
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L173
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L214
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L80


# [L-10] Redundant casting address(msg.sender)
msg.sender is already returning the address of the sender, hence, there is no need to cast it as `address(msg.sender)`

It is used in such case
```
(bool sent, ) = address(msg.sender).call{value: address(this).balance}( "");
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L124
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L110
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L84
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L63


# [L-11] `keccack256(abi.encodePacked())` Allows Hash Collision when used with more than one dynamic type
From the solidity documentation:
https://docs.soliditylang.org/en/v0.8.17/abi-spec.html?highlight=collisions#non-standard-packed-mode

> If you use keccak256(abi.encodePacked(a, b)) and both a and b are dynamic types, it is easy to craft collisions in the hash value by moving parts of a into b and vice-versa. More specifically, abi.encodePacked("a", "bc") == abi.encodePacked("ab", "c").


It is being used in six instances in file `Reth.sol`

For instance:
```
.getAddress(keccak256(abi.encodePacked("contract.address", "rocketTokenRETH")));
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L69
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L124
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L135
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L161
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L190
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L232


