## [L-01] Unable to remove derivatives

There is no functionality of the contract to remove a derivative from the index
fund. In the case where an incorrect derivative is deployed or an underlying
derivative is compromised and is no longer wanted to be used there is no way to
remove this from the contract.

Although this can be mitigated by setting the weight of the derivative to 0 this
will become useless storage on the contract and will increase the overall cost
of operations on the contract when it has to iterate through all derivatives.

### Mitigation

Implement a removeDerivative() function which will remove the derivative from
the contract. This will have implications on how derivatives are added the
contract since it uses the `derivativeCount` variable to create new entries in
the derivative mapping.

## [L-02] Gas Griefing is possible on external calls

Gas griefing is possible on external calls as the return data has to be stored
even if it is omitted, due to the EVM architecture:

```sol
(bool sent,) = address(msg.sender).call{value: ethAmountToWithdraw}("");
```

To mitigate this, perform a low level call with out and outdata size set to 0
so that the return data is not stored. 

```
@@ -108,7 +108,9 @@ contract SafEth is Initializable, ERC20Upgradeable, OwnableUpgradeable, SafEthSt
         uint256 ethAmountAfter = address(this).balance;
         uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;
         // solhint-disable-next-line
-        (bool sent,) = address(msg.sender).call{value: ethAmountToWithdraw}("");
+        assembly {
+            sent := call(gas(), caller(), ethAmountToWithdraw, 0, 0, 0, 0)
+        }
         require(sent, "Failed to send Ether");
         emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
     }
```



Check this tweet for more details:
https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19

## [L-03] Use Ownable2StepUpgradeable instead of OwnableUpgradeable

The `transferOwnership` function is used to change owners of the contract. This 
is using OZ's `OwnableUpgradeable` access-control contract.

There is a more secure contract called `Ownable2StepUpgradeable` which requires
the new owner to accept the ownership. This is safer and more secure as it
prevents ownership being sent to a bad address.

This is mostly applicable to the SafEth contract, but can be applied to the
derivatives, however the derivatives owners are unlikely to change.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L18
