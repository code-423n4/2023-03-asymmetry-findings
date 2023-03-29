### Non-Critical Items
| Number |Issues Details|
|:--:|:-------|
|[NC-01]| Inconsistent use of function rethAddress |
|[NC-02]| Inconsistent natspec for withdraw function |
|[NC-03]| _amount Parameter for ethPerDerivative |
|[NC-04]| Clean up arithmetic in Reth's deposit function |
|[NC-05]| Floating version pragma |

## NC-01 Inconsistent use of function rethAddress

In Reth.sol there are several instances of copy pasting the contents of the rethAddress() function rather than just using the function.

For example, starting at line 187:

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L187-L196

    address rocketTokenRETHAddress = RocketStorageInterface(
        ROCKET_STORAGE_ADDRESS
    ).getAddress(
        keccak256(
            abi.encodePacked("contract.address", "rocketTokenRETH")
        )
    );

where this function could be used instead:

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L66-L73

    function rethAddress() private view returns (address) {
        return
            RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
                keccak256(
                    abi.encodePacked("contract.address", "rocketTokenRETH")
                )
            );
    }

So it should be:

    address rocketTokenRETHAddress = rethAddress()

And really you could just set up the interface with rethAddress() as the parameter rather than caching the address it since it's only used once.

This whole code block

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

could be replaced by

    RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(
        rethAddress()
    );

The same issue also occurs within the poolPrice() function:

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L229-L235

    address rocketTokenRETHAddress = RocketStorageInterface( 
            ROCKET_STORAGE_ADDRESS
        ).getAddress(
                keccak256(
                    abi.encodePacked("contract.address", "rocketTokenRETH")
                )
            );

can be replaced by:

    address rocketTokenRETHAddress = rethAddress()

## NC-02 Inconsistent natspec for withdraw and deposit functions

The natspec for the withdraw and deposit functions in Reth.sol does not specify that the owner is the safEth contract.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L152-L155

    /**
        @notice - Deposit into derivative
        @dev - will either get rETH on exchange or deposit into contract depending on availability
     */ 

It does specify this in SfrEth.sol and WstEth.sol.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L90-L93

    /**
        @notice - Owner only function to Deposit into derivative
        @dev - Owner is set to SafEth contract
     */

Consider adding the @dev note about ownership to the Reth.sol functions for improved clarity and consistency

## NC-03 _amount Parameter for ethPerDerivative

The _amount parameter for the ethPerDerivative function is only used in the Reth.sol implementation. However, it's still present in the SfrxEth.sol and WstEth.sol implementations so that they conform to the IDerivative interface.

However, it may be helpful to add an @dev tag to the natspec comment explaining why the parameter is not used in those two implementations.

The parameter itself could also be commented out in the function definition:

    function ethPerDerivative(uint256 /*_amount*/) public view returns (uint256) 

## NC-04 Clean up arithmetic in Reth's deposit function

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L171-L174

    uint rethPerEth = (10 ** 36) / poolPrice();


    uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
        ((10 ** 18 - maxSlippage))) / 10 ** 18);
    
There's an extra set of parentheses around 10**18 - maxSlippage which makes it a little harder to follow. So it should be:

    uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
        (10 ** 18 - maxSlippage)) / 10 ** 18);

You could also substitute in the rethPerEth expression and cancel terms so that the formula for minOut is simply:

    uint256 minOut = msg.value * (10 ** 18 - maxSlippage) / poolPrice()

This also eliminates an instance of division before multiplication, so it would improve the calculation's precision.

## NC-05 Floating Version Pragma

Best practice is to set a concrete version pragma for deployment (i.e. =0.8.13 rather than ^0.8.13)

This ensures that you know which compiler version was actually used for deployment and allows you to use that version consistently across the codebase.
