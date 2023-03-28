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