# If there the owner adds wrong address in `addDerivative()` the contract needs to be redeployed
In the function [addDerivative](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182), more derivatives can be included(as much as the owner wants, currently 3). The issue is when an incorrect derivative address or address(0) is added, it causes a lot of issues. In all of the user interacted function there is a for loop that goes thru all of the derivatives and either deposits or withdraws. With incorrect address it will just fail and revert, but will also consume a lot of gas.The only way for this to be reverted is for the contract to be redeployed, costing even more gas. 
This issue could be mitigated with one simple function that can remove any unwanted derivatives.

# Function `ethPerDerivative()` is incorrectly implemented 
In the function `ethPerDerivative()` in [WstEth.sol/L86-L88](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86-L88) the function has a requirement of a `uint _amount`, but it is not implemented. It either needs to be inserted in `WStETH(WST_ETH).getStETHByWstETH(_amount)` or to be removed.

    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
    }

# Initializers should emit events 
In every contract there is an initialize function, but there isn't an event for it, my suggestion is to add an event `Initialized()` and make it so on initialization it emits the correct event with owner and variables.
In [SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L48)

    function initialize(
        string memory _tokenName,
        string memory _tokenSymbol
    ) external initializer {
        ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
        _transferOwnership(msg.sender);
        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
    +   emit Initialized(msg.sender,minAmount,maxAmount);
    }


# Imports should follow the solidity guidelines 
It is a simple change that follows the solidity style guide and makes everything easier for reading, understanding and auditing.
from:

    import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
    import "../interfaces/IWETH.sol";
    import "../interfaces/uniswap/ISwapRouter.sol";
    import "../interfaces/lido/IWStETH.sol";
    import "../interfaces/lido/IstETH.sol";
    import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
    import "./SafEthStorage.sol";
    import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
to:

    import {IWETH} from "../interfaces/IWETH.sol";
    import {SafEthStorage} from"./SafEthStorage.sol";
    import {IstETH} from "../interfaces/lido/IstETH.sol";
    import {IWStETH} from "../interfaces/lido/IWStETH.sol";
    import {ISwapRouter} from "../interfaces/uniswap/ISwapRouter.sol";
    import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
    import {OwnableUpgradeable} from "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
    import {ERC20Upgradeable} from"@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
