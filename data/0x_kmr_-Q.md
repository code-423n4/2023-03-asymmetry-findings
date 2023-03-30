GAS OPTIMIZATIONS:

### G[1] Use arithmetic operating instead using loop for calculating totalWeight adjustment 
[Link to github permalink](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175)
```
function adjustWeight(
    uint256 _derivativeIndex,
    uint256 _weight
) external onlyOwner {
+    uint256 oldWeight = weights[_derivativeIndex];
+    weights[_derivativeIndex] = _weight;
+    totalWeight = totalWeight - oldWeight + _weight;

-    weights[_derivativeIndex] = _weight;
-    uint256 localTotalWeight = 0;
-    for (uint256 i = 0; i < derivativeCount; i++)
-       localTotalWeight += weights[i];
-    totalWeight = localTotalWeight;
    emit WeightChange(_derivativeIndex, _weight);

}
```

The code above shows a function called `adjustWeight` that allows the owner to adjust the weight of a derivative. The function has been optimized to reduce gas consumption by eliminating the loop and instead, tracking the old weight, updating the new weight, and recalculating the total weight with a simple arithmetic operation.

This new code adds a variable `oldWeight` to store the original weight of the derivative before updating it. Then, the function updates the derivative's weight with the new weight provided by the owner and recalculates the total weight by subtracting the old weight and adding the new weight. This eliminates the need for a loop that calculates the total weight of all derivatives.

## G[2] Use arithmetic operating instead using loop for new totalWeight calculating
[Link to github permalink](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182-L195)
```
function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;

       // @remind
 gas/extra calcualtion
+      totalWeight = totalWeight + _weight;

-      uint256 localTotalWeight = 0;
-       for (uint256 i = 0; i < derivativeCount; i++)
-           localTotalWeight += weights[i];
-       totalWeight = localTotalWeight;
        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    }
```
Can be easily update `totalWeight = totalWeight + _weight;` since ` x = x + y` take less gas `x+=y`.
also there is no need of `for loop` to recalculate weight.

## G[3] checking for ethAmountBefore before entering loop can save a lot of gas if its value is 0.
[Link to github permalink](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138-L155)
```diff
 function rebalanceToWeights() external onlyOwner {
        uint256 ethAmountBefore = address(this).balance;
        for (uint i = 0; i < derivativeCount; i++) {
            if (derivatives[i].balance() > 0)
                derivatives[i].withdraw(derivatives[i].balance());
        }
        uint256 ethAmountAfter = address(this).balance;
        uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;
        // @remind

+       if ethAmountToRebalance > 0{
            for (uint i = 0; i < derivativeCount; i++) {
-                if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
+                if (weights[i] == 0) continue;
                uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                    totalWeight;
                // Price will change due to slippage
                derivatives[i].deposit{value: ethAmount}();
            }
+        }
        emit Rebalanced();
    }
```
Here, by adding `if ethAmountToRebalance > 0` condition, we can save a lot of gas by escaping from loop if `ethAmountToRebalance = 0` since `ethAmountToRebalance` been checked for every iteration.

## G[4] Unwanted calcualtion cause more gas
[Link to github permalink](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L211-L216
)
```diff
 function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        if (poolCanDeposit(_amount))
            return
                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
-       else return (poolPrice() * 10 ** 18) / (10 ** 18);
+       else return poolPrice();
    }
```
Since `poolPrice()` return `uint256`, we can directly cancel the `10**18 to 10**18`, this saves more gas.


## G[5] Functions guaranteed to revert_ when called by normal users can be marked payable.
If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

Instances:
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L168
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L185
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L205
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241


## G[5] .Setting the constructor to payable
You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

Instances:
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L38
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L33
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L27
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L24


## G[6] With assembly, .call (bool success) transfer can be done gas-optimized.
return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

Referencelink:https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19

Instances:
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L63
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L76
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L110
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L84
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L124


## G[7] .Using less than uint 32 are more gas ineffiecnt
The EVM works with 256bit/32byte words (debatable design decision). Every operation is based on these base units. If your data is smaller, further operations are needed to downscale from 256 bits to 8 bits, hence why you see increased costs.

Instances:

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L86


##  LOW CRITICAL ISSUES

## G[1]. It is recommended to divide with constant variables instead of numbers.
Instances:
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L94
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L75
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L174


## G[2] .Use a more recent version of solidity
In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

Instances :
In safEth.sol
pragma solidity ^0.8.13;

In Reth.sol
pragma solidity ^0.8.13;

In SfrxEth.sol
pragma solidity ^0.8.13;

In WstEth.sol
pragma solidity ^0.8.13;

## G[3] . Zero address check is the best practice.
  Validate against zero address to prevent gas waste as well as contract lock.
Instances:
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182


## G[4] .Floating pragma.
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

Instances:
In safEth.sol
pragma solidity ^0.8.13;

In Reth.sol
pragma solidity ^0.8.13;

In SfrxEth.sol
pragma solidity ^0.8.13;

In WstEth.sol
pragma solidity ^0.8.13;


