
##### [L-01] Use Ownable2StepUpgradeable instead of OwnableUpgradeable contract
[SfrxEth.sol#L13](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L13)
[WstEth.sol#L12](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L12)
[Reth.sol#L19](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L19)

transferOwnership function is used to change Ownership from OwnableUpgradeable.sol.

There is another Openzeppelin Ownable contract (Ownable2StepUpgradeable.sol) has transferOwnership function , use it is more secure due to 2-stage ownership transfer.

[Ownable2StepUpgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol)

##### [L-02] Prevent division by 0
In the unstake() function precautions are not being taken for not dividing by 0, this will revert the code.
The above function can be called with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero.

[SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L116)

In the above calculation `safEthTotalSupply` can be 0 and there is no check for the value being greater than 0.


##### [L-03] Gas griefing/theft is possible on unsafe external call
return data (bool success,) has to be stored due to EVM architecture, if in a usage like below, ‘out’ and ‘outsize’ values are given (0,0).
Thus, this storage disappears and may come from external contracts a possible Gas griefing/theft problem is avoided

```
SafEth.sol :

        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
            ""
        );

        // The above check can be updated to the below format
        assembly {                                    
            sent := call(gas(), address(msg.sender), ethAmountToWithdraw, 0, 0)
        }
```

There are 5 instances of this particular usage

[SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L124)

[SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L84)

[Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L110)

[WstEth.sol#L63](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L63)

[WstEth.sol#L76](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L76)

### [L-04] unused import 

Please remove the unused import in SafEth.sol
```
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```
IERC20 is imported, but it is not being used anywhere in the code.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L4

### [L-05] Missing events for critical arithmetic parameters

It would be better if we can add a event for setMaxSlippage() function in Reth.sol and SfrxEth.sol since maxSlippage is used in deposit() and withdraw() functions in the above two contracts respectively.

[Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L58)

[SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51)

setMaxSlippage() does not emit an event, so it is difficult to track changes in the value of maxSlippage off-chain.