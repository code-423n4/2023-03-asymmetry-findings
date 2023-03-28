In the file Reth.sol there are many hashs, which are calculated dynamically, but can be initialized as consts:
keccak256(abi.encodePacked("contract.address", "rocketTokenRETH"))
keccak256(abi.encodePacked("contract.address", "rocketDepositPool"))
keccak256(abi.encodePacked("contract.address","rocketDAOProtocolSettingsDeposit"))

It will reduce function gas costs

----
In contacts where loops are used contract can be optimized by setting i++ increment within unchecked operation
unchecked {
            i += 1;
        }

----

`derivativeCount` is a derived variable be which can be received from derivatives.length
It's bad practice to store contract-derived results.
A good practice is to create view function derivativeCount, as follows

```
function derivativeCount() private view returns(uint256) {
 return derivatives.length;
}
```
and we can use calls for these functions everywhere. In other functions, we can reduce the manual increment of the derivativeCount variable, which will decrease gas cost for external function execution
function adjustWeight can be optimized as follow
```
    function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
        uint256 oldWeight = weights[_derivativeIndex];
        weights[_derivativeIndex] = _weight;
        totalWeight = totalWeight - oldWeight + _weight;
        emit WeightChange(_derivativeIndex, _weight);
    }
```