## Table of Contents

- [L-01] Use Ownable2StepUpgradeable instead of OwnableUpgradeable contract
- [L-02] Max Slippage should have a maximum cap to prevent users from losing value when depositing / withdrawing their derivatives
- [L-03] Make sure minAmount is lesser than maxAmount and vice versa when changing the values
- [L-04] Unbounded loops may caus DoS if there is too many derivatives in the protocol
- [L-05] Derivatives cannot be deleted from the contract
- [L-06] Solidity version used (0.8.13) has known issues
- [L-07] Hardcoded fee can be refactored
- [L-08] Swapping in Reth.sol may be sub-optimal
- [L-09] Division before multiplication should be avoided
- [N-01] Non-library/interface files should use fixed compiler versions, not floating ones
- [N-02] abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
- [N-03] For modern and more readable code; update import usages
- [N-04] Zero values should be checked for best practice


### [L-01] Use Ownable2StepUpgradeable instead of OwnableUpgradeable contract

transferOwnership function is used to change Ownership from Ownableupgradeable.sol. There is another Openzeppelin Ownable contract (Ownable2Stepupgradeable.sol) has transferOwnership function , use it is more secure due to 2-stage ownership transfer. 

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L9

### [L-02] Max Slippage should have a maximum cap to prevent users from losing value when depositing / withdrawing their derivatives

In the derivatives contracts, eg WstEth.sol, the slippage is set at 1%. However, the owner can change the slippage anytime, which may lead to unintended swapping behaviours if slippage is too high. Set a hard cap so users can get a certain minimum value back when depositing / withdrawing funds.  

```
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
	  require(_slippage < 1e18, "Too high slippage")
        maxSlippage = _slippage;
    }
```

The maxSlippage variable is used to calculate minOut. If maxSlippage is too high (ie 100%) , then minOut will be 0. The user may deposit 10 ETH and get 0 back.

```
        uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
```

### [L-03] Make sure minAmount is lesser than maxAmount and vice versa when changing the values

Whenever users deposit ETH to get SafETH in the SafEth.sol contract, the function `stake()` checks if msg.value is > minAmount and < maxAmount. However, when the owner is changing the minmax values, minAmount is not checked against maxAmount. There can be a case where minAmount is > maxAmount or otherwise.

```solidity
    function setMinAmount(uint256 _minAmount) external onlyOwner {
+       require(_minAmount != 0, "Zero Value")
+	  require(_minAmount < maxAmount, "Wrong Value")
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }


    /**
        @notice - Owner only function that sets the maximum amount a user is allowed to stake
        @param _maxAmount - amount to set as maximum stake value
    */
    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
+       require(_maxAmount != 0, "Zero Value")
+	  require(_maxAmount > minAmount, "Wrong Value")
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
    }
```

If maxAmount is greater than minAmount, then the staking function does not work anymore. Will be good practice to have these checks and also 0 value checks

### [L-04] Unbounded loops may caus DoS if there are too many derivative contracts in the protocol

In SafEth.sol, when a user stakes his ETH to get SafETH, the contract loops over every derivative contracts and deposits the appropriate weight into different protocols. The process from ETH to SafETH is quite long and requires many checks and transfers, and may reach gas limit before every process finishes. Right now, there is only 3 derivative contracts, so the function may be able to handle the loops. If the protocol decides to add more derivatives, then there may be a time where the stake and unstake function continuously fails due to exceeding gas limits. 

Consider checking the limit of the derivativeCount and having an upper bound limit. Also recommend having emergency withdrawals for each derivative.

```
///@---audit when the derivativeCount is too large, this loop will always fail and users cannot stake/unstake anymore
        for (uint256 i = 0; i < derivativeCount; i++) {
            // withdraw a percentage of each asset based on the amount of safETH
            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
            if (derivativeAmount == 0) continue; // if derivative empty ignore
            derivatives[i].withdraw(derivativeAmount);
        }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113-L119

### [L-05] Derivatives cannot be deleted from the contract

In SafEth.sol, the owner can add derivatives to populate the project with more attractive APRs. Right now, there is frax, rocket pool and Lido, but the owner can add more projects by calling addDerivative() and adjusting the weight. If the protocol decides not to use a project anymore, then they can set the weight to zero. 

However, the protocol does not have an option to removeDerivative. It is recommended to remove the project if it is not being used instead of simply setting the weight to zero to save gas because everytime a user stakes/unstakes into the project, the obsolete project with zero weight will still be looped, which wastes a small amount of gas.
```
            if (weight == 0) continue;
```

It is better to have a function that removes the derivative completely.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182-L195

### [L-06] Solidity version used (0.8.13) has known issues

1. Vulnerability related to ABI-encoding.

ref : https://blog.soliditylang.org/2022/05/18/solidity-0.8.14-release-announcement/

2. Vulnerability related to 'Optimizer Bug Regarding Memory Side Effects of Inline Assembly'

ref : https://blog.soliditylang.org/2022/06/15/solidity-0.8.15-release-announcement/

Solidity versions 0.8.13 and 0.8.14 are vulnerable to a recently reported optimizer bug related to inline assembly. Solidity 0.8.15 has been released with a fix. This bug only occurs under very specific conditions: the legacy optimizer must be enabled rather than the IR pipeline (true for the current project configuration), and the affected assembly blocks must not refer to any local Solidity variables. 

It's worth being aware of this vulnerabilities although both does not really relate as far as I know (protocol does not use abi.encode or have nested arrays, and openzeppelin files used do not contain inline assembly). Still, consider upgrading to Solidity 0.8.15.

### [L-07] Hardcoded fee can be refactored

This is more of refactoring than an issue. Usually hardcoded fees are problematic because it is not flexible and the LP in the pool may be too little. This protocol uses Uniswap 0.05% fee Reth/Weth liquidity pool, as seen in the code


```
            uint256 amountSwapped = swapExactInputSingleHop(
                W_ETH_ADDRESS,
                rethAddress(),
                500,
                msg.value,
                minOut
            );
```


In this case, there is [5M liquidity](https://www.geckoterminal.com/eth/pools/0xa4e0faa58465a2d369aa21b3e42d43374c6f9613) in the Reth/Weth Uniswap 0.05% pool and there is only one hardcoded fee, so its not much of a problem. I just want to share how another protocol does the refactoring of the hardcoded fees, making swaps more flexible.

Recommended Fee Factoring: https://github.com/sturdyfi/code4rena-may-2022/pull/12/files



### [L-08] Swapping in Reth.sol may be sub-optimal

The Reth pool uses the Weth/Reth 0.05% fee pool to swap between weth and reth. I recommend using the balancer pool to swap instead as it has 80M liquidity compared to Uniswap's 5M liquidity, which is better for larger swaps. Balancer pool also has a lower fee percent at 0.04%

https://www.geckoterminal.com/eth/pools/0x1e19cf2d73a72ef1332c882f20534b6519be0276

### [L-09] Division before multiplication should be avoided

When depositing into the rocket pool, the ETH is first converted into reth. At one junture, minOut is calculated (which indicates the minimum output from a given amount of ETH deposited). In order to prevent any truncation edge cases, it is better to multiply everything before dividing.

```
            uint rethPerEth = (10 ** 36) / poolPrice();


            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
                ((10 ** 18 - maxSlippage))) / 10 ** 18);
```
```
-	minOut = ((((rethPerEth * msg.value) / 10 ** 18) * ((10 ** 18 - maxSlippage))) / 10 ** 18);
+	minOut = (((rethPerEth * msg.value) * ((10 ** 18 - maxSlippage))) / 10 ** 36);
```

### [N-01] Non-library/interface files should use fixed compiler versions, not floating ones

Issue relates to all contracts in scope.

```
- pragma solidity ^0.8.13;
+ pragma solidity 0.8.13;
``` 

### [N-02] abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()

Use abi.encode() instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). "Unless there is a compelling reason, abi.encode should be preferred". If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.

```
                keccak256(
-                    abi.encodePacked("contract.address", "rocketDepositPool")
+                    abi.encode("contract.address", "rocketDepositPool")
                )
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L124-L126

### [N-03] For modern and more readable code; update import usages

Exact functions should be imported. Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

```
import "../../interfaces/IDerivative.sol";
import "../../interfaces/frax/IsFrxEth.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../../interfaces/curve/IFrxEthEthPool.sol";
import "../../interfaces/frax/IFrxETHMinter.sol";
```

import {contract1 , contract2} from "filename.sol";

An example from Art Gobblers: 

```
import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";
```

### [N-04] Zero values should be checked for best practice

When changing any values, zero values should be checked for good practice. For example, the setting the maxAmount should check for zero value.

```
    /**
        @notice - Owner only function that sets the maximum amount a user is allowed to stake
        @param _maxAmount - amount to set as maximum stake value
    */
    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
+       require(_maxAmount != 0, "Zero Value")
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
    }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L213-L226