## 1. `rethAddress()` CAN BE USED TO GET THE `rocketTokenRETH` CONTRACT ADDRESS INSTEAD OF DUPLICATING THE SAME CODE SNIPPET

The `Reth.sol` contract uses the `rethAddress()` private function to return the `rocketTokenRETH` contract address.

    function rethAddress() private view returns (address) {
        return
            RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
                keccak256(
                    abi.encodePacked("contract.address", "rocketTokenRETH")
                )
            );
    }

But in the `deposit` and `poolPrice` functions of the `Reth` contract, the same code snippet is duplicated to find the `rocketTokenRETH` contract address.

            address rocketTokenRETHAddress = RocketStorageInterface(
                ROCKET_STORAGE_ADDRESS
            ).getAddress(
                    keccak256(
                        abi.encodePacked("contract.address", "rocketTokenRETH")
                    )
                );
            RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(
                rocketTokenRETHAddress
            );

But the `rethAddress()` private function can be directly called to find the `rocketTokenRETH` contract address instead of duplicating the same code snippet in the `deposit` and `poolPrice` functions. 
Above code can be updated and made simpler in the `deposit` function as follows:

            RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(rethAddress());
			
This will save on the deployment gas of the `Reth.sol` contract.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L187-L196
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L229-L235

## 2. CALLDATA VARIABLE SHOULD BE USED TO EMIT THE EVENT INSTEAD OF USING THE STATE VARIABLE.

It is gas efficient to use the calldata variable as the argument to `ChangeMinAmount` event instead of using the state variable.
This will save an extra `Gcoldsload - 2100 gas`.

    function setMinAmount(uint256 _minAmount) external onlyOwner {
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }

The `emit ChangeMinAmount(minAmount)` event can be coded as follows to save gas:

       emit ChangeMinAmount(_minAmount);

`_minAmount` is calldata variable which is passed into the function as an input parameter.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214-L217

There are 3 more instances of this issue:
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223-L226
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232-L235
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L241-L244

## 3. CAN DECLARE THE VARIABLE OUTSIDE THE `for` LOOP TO SAVE GAS

Consider making the stack variables before the `for` loop which will save gas.

        for (uint256 i = 0; i < derivativeCount; i++) {
            // withdraw a percentage of each asset based on the amount of safETH
            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
            if (derivativeAmount == 0) continue; // if derivative empty ignore
            derivatives[i].withdraw(derivativeAmount);
        }
		
In the above code snippet, stack variable `uint256 derivativeAmount` can be declared outside the `for` loop.
			
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113-L119

There are 2 more instances of this issue:

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L85-L98
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L149

## 4. SECOND CALL TO THE `derivatives[i]` INSIDE THE `for` LOOP CAN BE CACHED TO SAVE GAS.

In the `safEth.sol` contract, `derivatives` mapping is used to iterate through the derivative contracts in a `for` loop and call thier respective functions.
But inside the same `for` loop, `derivatives[i]` has been called multiple times.

        for (uint i = 0; i < derivativeCount; i++)
            underlyingValue +=
                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                    derivatives[i].balance()) /
                10 ** 18;

The `derivatives[i]` can be cached to a memory variable, and the memory variable can be used inside the `for` loop instead. Following code snippet depicts this change:

        for (uint i = 0; i < derivativeCount; i++){
			IDerivative derivative = derivatives[i];
            underlyingValue +=
                (derivative.ethPerDerivative(derivative.balance()) *
                    derivative.balance()) /
                10 ** 18;
		}


This will save gas since `derivatives[i]` will be called from storage only the first time, and remaining calls will use the memory variable.
This will save an extra `Gcoldsload - 2100 gas`, every time memory variable is used in place of the `derivatives[i]` in the for loop.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71-L75

There are 2 more instances of this issue:

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113-L119
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140-L143

## 5. SECOND CALL TO THE `derivatives[i].balance()` INSIDE THE `for` LOOP CAN BE CACHED TO SAVE GAS.

In the `SafEth.sol` contract, the `derivatives[i].balance()` is used to get the derivative balance from each of the index fund derivatives.
And `derivatives[i].balance()` is used inside the `for` loops to iterate through all the derivatives of the index fund.

        for (uint i = 0; i < derivativeCount; i++) {
            if (derivatives[i].balance() > 0)
                derivatives[i].withdraw(derivatives[i].balance());
        }

But inside the `for` loop `derivatives[i].balance()` is called multiple times. 
Hence rather than calling the `balance()` public function of the respective derivatives contracts multiple times inside the `for` loops, the return value of `derivatives[i].balance()` can be cached to stack variable and read from it.
The updated code snippet is shown below:

        for (uint i = 0; i < derivativeCount; i++) {
		uint256 derivativeBalance = derivatives[i].balance();
            if (derivativeBalance > 0)
                derivatives[i].withdraw(derivativeBalance);
        }

This will save gas since it replaces an external call to a public function with a gas efficient memory load of the variable.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140-L143

There is 1 more instance of this issue:

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71-L75
