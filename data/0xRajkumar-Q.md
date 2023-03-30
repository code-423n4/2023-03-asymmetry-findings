# 1.Wrong index passed "DerivativeAdded()"
## In SafEth contract we have "addDerivative()" function which emits event "DerivativeAdded()" and this event expect index of newly derivative. 
## In "addDerivative()" function we always add new derivative and increases derivativeCount by one and this same variable is always passed into event "DerivativeAdded()" that's why we always pass wrong index which is index of next derivative. 
#### References
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L194
#### Mitigation step
We should pass derivativeCount-- instead of derivativeCount
````
    function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;

        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit DerivativeAdded(_contractAddress, _weight, derivativeCount--);
    }

````
# 2.Contract Owner Has Too Many Privileges
## The owner of the contracts has too many privileges relative to standard users. The consequence is disastrous if the contract owner’s private key has been compromised. And, in the event the key was lost or unrecoverable, no implementation upgrades and system parameter updates will ever be possible. For a project this grand, it increases the likelihood that the owner will be targeted by an attacker, especially given the insufficient protection on sensitive owner private keys. The concentration of privileges creates a single point of failure; and, here are some of the incidents that could possibly transpire:
### Transfer ownership and mess up with all the setter functions, hijacking the entire protocol.

#### Consider:

splitting privileges (e.g. via the multisig option) to ensure that no one address has excessive ownership of the system,
clearly documenting the functions and implementations the owner can change,
documenting the risks associated with privileged users and single points of failure, and
ensuring that users are aware of all the risks associated with the system.
# 3.Function Calls in Loop Could Lead to Denial of Service
## Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to the gas limitations or failed transactions. Here are some of the instances entailed:
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113
#### Consider 
Bounding the loop where possible to avoid unnecessary gas wastage and denial of service.
# 4.Loss of precision
## In SafEth.sol we have "Stake()" function in which we calculate underlyingValue and preDepositPrice as you can see here
````
        for (uint i = 0; i < derivativeCount; i++)
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                    derivatives[i].balance()) /
                10 ** 18;

        uint256 totalSupply = totalSupply();
        uint256 preDepositPrice; // rice of safETH in regards to ETH
        if (totalSupply == 0)
            preDepositPrice = 10 ** 18; // initializes with a price of 1
        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

````
### To underlyingValue calculate we divide ethPerDerivative*balance with 1e18 and to calculate preDepositPrice we again multiply with 1e18 so here when we divide with 1e18 causes loss of precision instead we should not divide and multiply with 1e18 we will have precise preDepositPrice.
#### Example
````
ethPerDerivative = 1232427936278277338
balance = 1132454354667767865
totalsupply = 1029343465645635565

Without precision loss = 1355882103333836962
With precision loss = 1355882103333836961
````
# 5.Use a more recent version of solidity
## Current version of solidity is v0.8.19 
#### References
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L2
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L2
# No check in "setMaxSlippage()" function
## Using "setMaxSlippage()" owner can set max slippage but function should not allow max slippage more than some limit because small owner mistake can cause loss of user funds.
#### References
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58
# 6.Data location must be “calldata” for parameter in external function
## According to official solidity documentation https://docs.soliditylang.org/en/v0.5.0/050-breaking-changes.html#explicitness-requirements
````
Note that external functions require parameters with a data location of calldata.
````
#### This is violated here
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L48
#### Advantage
We can save some gas also by marking it as calldata