## 1. USE `ownable2stepUpgradeable.sol` instead of `ownableUpgradeable.sol`

According to the `Natspec` the owner of the `SfrxEth.sol`, `Reth.sol` and `WstEth.sol` are expected to be the `SafEth` contract. 
In the `initialize()` function of the three derivative contracts the owner is set as the `_owner` which is passed in as an input address parameter.
The `Initialize` function then uses the `OwnableUpgradeable` function `_transferOwnership(_owner)` to set the contract owner as the `_owner` address.

    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }

Owner is a key role in each of the three derivative contracts. 
It controls `onlyOwner` modifier applied functions of the contracts, which enables only the owner to execute `withdraw`, `deposit` and `setMaxSlippage` functionalities.
Above mentioned functions are the core functions which determines the functioning of the protocol.
Hence setting the owner address should be done securely since errorneous owner address could prompt redeployment of the contract. 

Hence it is recommened to use the `ownable2stepUpgradeable.sol` instead of `ownableUpgradeable.sol` as a best practice in setting the owner of the contract.

In the `ownable2stepUpgradeable.sol` the ownership is transfered in two steps:
	1. The new owner is set as the pending owner in the `transferOwnership` function.
	2. The new owner accepts the ownership calling the `acceptOwnership` function of the `ownable2stepUpgradeable.sol` contract.

Hence there is no possibility for an erroneous address to be set the new owner, since new owner himself should accept the ownership by calling the `acceptOwnership` as `msg.sender`.

Above code should be updated as below: (Here `transferOwnership` is a public function in the `ownable2stepUpgradeable.sol` contract).

    function initialize(address _owner) external initializer {
        transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }
	
And later the new owner should call the `acceptOwnership()` function of the `ownable2stepUpgradeable.sol` contract, for actual transfer of the ownership.


https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L42-L45

There are 2 more instances of this issue:

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L36-L39
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L33-L36


## 2. CHECK FOR `address(0)` WHEN SETTING THE `_owner` ADDRESS OF THE DERIVATIVE CONTRACTS

The `initialize` function of each of the derivative contracts accept the `_owner` input address parameter.
Then sets the owner of the contract as the `_owner` parameter using the following command.

        _transferOwnership(_owner);

But the `_owner` is never checked for the `address(0)`. 
Since the owner of the contract controls the `onlyOwner` modifier applied important functions of the protocol, it is recommended to be extra cautious in setting the owner of the contract.
Hence as a best practice it is recommended to check for address(0) before the transfer of the ownership as follows:

    function initialize(address _owner) external initializer {
		require(_owner != address(0), "Zero Address Passed In");
        _transferOwnership(_owner);
        maxSlippage = (1 * 10 ** 16); // 1%
    }

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L42-L45

There are 2 more instances of this issue:

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L36-L39
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L33-L36

## 3. `_amount` INPUT PARAMETER PASSED INTO THE `ethPerDerivative` FUNCTION IS NEVER USED.

The `SfrxEth` contract and `WstEth` contract use the `ethPerDerivative` function to calculate the price per derivative in terms of `ETH`.
The `ethPerDerivative` function takes in an input parameter as shown below in its function signature.

    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
	
But this input parameter is never used inside the function implementation logic. 
It seems this function signature is used, in order to follow the same function signature format since it is used by the `SafEth` contract.
Since the `Reth` contract makes use of this `_amount` input parameter in its `ethPerDerivative` function it is used as the same in `SfrxEth` contract and `WstEth` contract as well.

Since `SfrxEth` contract and `WstEth` contract does not use the `_amount` input parameter in thier `ethPerDerivative` function implemenation, the parameter name can be commented as below:

    function ethPerDerivative(uint256 /*_amount*/) public view returns (uint256) { 

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86

There is 1 more instance of this issue:

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L111

## 4. RECOMMENDED TO USE A REASONABLE VALUE FOR `sqrtPriceLimitX96` INSTEAD OF `0`

In the `Reth` contract `swapExactInputSingleHop` function uses the `uniswap` to swap the `WETH` tokens to `Reth` tokens.
The function uses the `ExactInputSingleParams` struct to set the required parameters for the uniswap swap transaction.

In these parameters the `sqrtPriceLimitX96` parameter is set to `0`.
This parameter, if set, is used to execute the transaction with in a certain price range of the token.
Since this parameter is 0 and not set, it means the swap can happen at any price of the token.
This could be disadvantageous to the user of the protocol when `staking` funds during volatile market conditions.
Because the swap trade can be executed at an unfavourable price for the user.

Hence it is recommended to apply a reasonable `sqrtPriceLimitX96` parameter value for the swap trade between `WETH` and `Reth`. 
So that trade will only execute as long as the price of the token is with in certain price range of the token calculated based on `sqrtPriceLimitX96` value.
This will save the user of any unfavourable swap occurence, since if the price range is exceeded then the transaction will revert.

        ISwapRouter.ExactInputSingleParams memory params = ISwapRouter
            .ExactInputSingleParams({
                tokenIn: _tokenIn,
                tokenOut: _tokenOut,
                fee: _poolFee,
                recipient: address(this),
                amountIn: _amountIn,
                amountOutMinimum: _minOut,
                sqrtPriceLimitX96: 0
            });
			
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L99

## 5. USE `uint256` INSTEAD OF `uint` TO MAKE THE VARIABLE DECLARATIONS FOLLOW THE SAME FORMAT.

The `uint256` and `uint` data types are the same in terms of size of the data type.
But it can be confusing to the auditor and developers when `uint256` and `uint` is used to declare variables interchangeably.

In the `SafEth` contract there are multiple occassions where `uint` is used to declare the variables. 

        for (uint i = 0; i < derivativeCount; i++)
		
`uint256` can be used instead.

Hence in the `safEth` contract `uint256` and `uint` are used to interchangeably to declare variables and it is confusing.

Hence it is recommended to use `uint256` when declaring variables instead of using `uint`.
This will enchance the code readability.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71

There are 5 more instances of this issue:

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L92
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L203-L204
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L26-L32

## 6. `safEthTotalSupply` SHOULD BE CHECKED FOR `0` VALUE

In the `unstake` function of the `SafEth.sol` contract, `safEthTotalSupply` value is calculated as the total supply of `safEth` tokens.
The `safEthTotalSupply` value is then used to calculate the unstaking derivative amount for each derivative type as shown below:

            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
				
But the `safEthTotalSupply` is not checked for `0` value, before the above calculation is performed.

It is recommended to check for `safEthTotalSupply != 0` condition before the calculation for `derivativeAmount` is performed.
If not there is a possibility of division by zero if the `safEth` token amount in the contract is zero.

Since the `unstake` function can be called by any user, above division by zero can occur.
Hence it is better to revert the transaction if the `safEthTotalSupply` value is zero.

Hence add the following condition before the `for` loop in the `unstake` function as below:

        uint256 safEthTotalSupply = totalSupply();
        uint256 ethAmountBefore = address(this).balance;
		
		require(safEthTotalSupply != 0, "safEthTotalSupply is zero");

        for (uint256 i = 0; i < derivativeCount; i++) {
            // withdraw a percentage of each asset based on the amount of safETH
            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
            if (derivativeAmount == 0) continue; // if derivative empty ignore
            derivatives[i].withdraw(derivativeAmount);
        }

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L110-L119

## 7. ADD `address(0)` CHECK FOR `_contractAddress` WHEN ADDING NEW DERIVATIVE VIA `addDerivative()` FUNCTION.

In the `SafEth.sol` contract, `addDerivative()` function is used to add a new derivative to the protocol.
And the `addDerivative()` function takes in two parameters `_contractAddress` and `_weight`.

    function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;
		
		...
		
	}

The added derivative via the `addDerivative()` function will later be used to stake the deposited `Eth` by the users.
Hence it is a must to ensure that the `_contractAddress`, the address of the derivative is a valid address.

But the `address(0)` validation check is not performed on the `_contractAddress` in the `addDerivative()` function.
Hence `address(0)` can be added to the `derivatives` mapping via `addDerivative()` function.
This could lead to `stake` function calling the `deposit` function on the address(0) when trying to stake user `Eth`.

This will break the protocol functionality since the transaction will revert.

Hence it is recommended to check for `address(0)` inside the `addDerivative()` function as below:

    function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
		require(_contractAddress != address(0), "Zero address passed in");
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;
		
		...
		
	}
	
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-L188

## 8. `OwnableUpgradeable` IS INHERITED FROM BUT NOT INITIALIZED IN THE `initialize` FUNCTION.

The `safEth`, `Reth`, `WstEth` and `SfrxEth` contracts inherit from the `OwnableUpgradeable` contract.
But the `OwnableUpgradeable` contract is not initialized by calling the `__Ownable_init()` function inside the `initialize()` function of the above mentioned contracts.

    function initialize(
        string memory _tokenName,
        string memory _tokenSymbol
    ) external initializer {
        ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
        _transferOwnership(msg.sender);
        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
    }

Hence even though the `OwnableUpgradeable` contract is used its not initialized properly.

It is recommended to call the `__Ownable_init()` function inside the `initialize` functions of the `safEth`, `Reth`, `WstEth` and `SfrxEth` contracts.
This will initialize the `OwnableUpgradeable` contract properly.

Hence above code snippet should be changed as follows:

    function initialize(
        string memory _tokenName,
        string memory _tokenSymbol
    ) external initializer {
        ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
		__Ownable_init();
        _transferOwnership(msg.sender);
        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
    }

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L48-L56

There are 3 more instances of this issue:

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L42-L45
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L36-L39
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L33-L36

## 9. THE `Natspec` COMMENT IS WRONG FOR THE `adjustWeight()` FUNCTION.

The `adjustWeight()` function is used to change the `weight` of a derivative.
But the `Natspec` comment says the following in its `@notice` tag. 

        //@notice - Adds new derivative to the index fund
		
Which says `adjustWeight()` function is used to add new derivative to the index fund. But that is not the case.
Infact the `addDerivative()` function is used for the above purpose.

Hence the above comment should be corrected as follows:

        //@notice - Changes weight of a given derivative index and updates the totalWeight
		
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L158