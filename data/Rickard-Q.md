# [L-01] Use `safeTransferOwnership` instead of `_transferOwnership` function
## Lines of code
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L34](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L34)     
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L53](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L53)       
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L43](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L43)      
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L37](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L37)
## Vulnerability details
`_transferOwnership` function is used to change Ownership from `OwnableUpgradeable.sol`.     
     
Use a 2 structure `_transferOwnership` which is safer. `safeTransferOwnership`, use is more secure due to 2-stage ownership transfer.
### Recommended Mitigation Steps
Use [Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol).

# [L-02] Lack of checks `address(0)`
## Lines of code
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L93-L94](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L93-L94)
## Vulnerability details
The following methods have a lack of checks if the received argument is an address, it’s good practice in order to reduce human error to check that the address specified in the constructor or initialize is different than `address(0)`.

# [L-03] Missing ReEntrancy Guard to withdraw function
## Lines of code
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107-L114](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107-L114)     
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60-L88](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60-L88)      
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56-L67](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56-L67)
## Vulnerability details
`Reth.sol`/`SfrxEth.sol`/`WstEth.sol` contract has no Re-Entrancy protection in `withdraw()` function.
````solidity
contracts/SafEth/derivatives/Reth.sol
    function withdraw(uint256 amount) external onlyOwner {
        RocketTokenRETHInterface(rethAddress()).burn(amount);
        // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
        require(sent, "Failed to send Ether");
    }

contracts/SafEth/derivatives/SfrxEth.sol
    function withdraw(uint256 _amount) external onlyOwner {
        IsFrxEth(SFRX_ETH_ADDRESS).redeem(
            _amount,
            address(this),
            address(this)
        );
        uint256 frxEthBalance = IERC20(FRX_ETH_ADDRESS).balanceOf(
            address(this)
        );
        IsFrxEth(FRX_ETH_ADDRESS).approve(
            FRX_ETH_CRV_POOL_ADDRESS,
            frxEthBalance
        );


        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
            (10 ** 18 - maxSlippage)) / 10 ** 18;


        IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).exchange(
            1,
            0,
            frxEthBalance,
            minOut
        );
        // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
        require(sent, "Failed to send Ether");
    }
    
contracts/SafEth/derivatives/WstEth.sol
    function withdraw(uint256 _amount) external onlyOwner {
        IWStETH(WST_ETH).unwrap(_amount);
        uint256 stEthBal = IERC20(STETH_TOKEN).balanceOf(address(this));
        IERC20(STETH_TOKEN).approve(LIDO_CRV_POOL, stEthBal);
        uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
        IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);
        // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
        require(sent, "Failed to send Ether");
    }
````
[https://www.paradigm.xyz/2021/08/the-dangers-of-surprising-code](https://www.paradigm.xyz/2021/08/the-dangers-of-surprising-code)
### Recommended Mitigation Steps
Use Openzeppelin or Solmate Re-Entrancy pattern.     
Here is a example of a re-entrancy guard:
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;
contract ReEntrancyGuard {
    bool internal locked;
    modifier noReentrant() {
        require(!locked, "No re-entrancy");
        locked = true;
        _;
        locked = false;
    }
}
````
# [L-04] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from `uint256`
## Vulnerability details
In the protocol, there are function's values with the risk of being downcasted.

## Recommended Mitigation Steps
Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from `uint256`.

# [L-05] `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will prevent [hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456) => 0x0...1230...456)`. "Unless there is a compelling reason, abi.encode should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` instead. If all arguments are strings and or bytes, `bytes.concat()` should be used [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).     
         
Instances (6):
````solidity
contracts/SafEth/derivatives/Reth.sol
67        return
68                    RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
69                        keccak256(
70                            abi.encodePacked("contract.address", "rocketTokenRETH")
71                        )
72                    );

124        keccak256(
125                            abi.encodePacked("contract.address", "rocketDepositPool")
126                        )
127:                    );

135        keccak256(
136                            abi.encodePacked(
137                                "contract.address",
138                                "rocketDAOProtocolSettingsDeposit"
139                            )
140:                        )

161        keccak256(
162                            abi.encodePacked("contract.address", "rocketDepositPool")
163                        )
164:                    );

190        keccak256(
191                                abi.encodePacked("contract.address", "rocketTokenRETH")
192                            )
193:                        );

232        keccak256(
233                            abi.encodePacked("contract.address", "rocketTokenRETH")
234:                        )
````


# [N-01] Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions
## Lines of code
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L15-L20](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L15-L20)     
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L19](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L19)        
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L13](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L13)      
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L12](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L12)
## Vulnerability details
See this [link](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.      
      
There is 4 instances of this issue:
````solidity
contracts/SafEth/SafEth.sol
15        contract SafEth is
16            Initializable,
17            ERC20Upgradeable,
18            OwnableUpgradeable,
19            SafEthStorage
20:        {

contracts/SafEth/derivatives/Reth.sol
19:     contract Reth is IDerivative, Initializable, OwnableUpgradeable {

contracts/SafEth/derivatives/SfrxEth.sol
13:     contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {

contracts/SafEth/derivatives/WstEth.sol
12:     contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
````

# [N-02] Lack of event emission after critical `initialize()` function
To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the `initialize()` function:
````solidity
4 results - 4 files
contracts/SafEth/derivatives/WstEth.sol
33        function initialize(address _owner) external initializer {
34                _transferOwnership(_owner);
35                maxSlippage = (1 * 10 ** 16); // 1%
36:            }

contracts/SafEth/derivatives/Reth.sol
42        function initialize(address _owner) external initializer {
43                _transferOwnership(_owner);
44                maxSlippage = (1 * 10 ** 16); // 1%
45:            }

contracts/SafEth/derivatives/SfrxEth.sol
36        function initialize(address _owner) external initializer {
37                _transferOwnership(_owner);
38                maxSlippage = (1 * 10 ** 16); // 1%
39:            }

contracts/SafEth/SafEth.sol
48        function initialize(
49                string memory _tokenName,
50                string memory _tokenSymbol
51            ) external initializer {
52                ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
53                _transferOwnership(msg.sender);
54                minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
55                maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
56:            }
````

# [N-03] Two Steps Verification before Transferring Ownership
## Vulnerability details
The following contracts have a function that allows them an admin to change it to a different address. If the admin accidentally uses an invalid address for which they do not have the private key, then the system gets locked.
It is important to have two steps admin change where the first is announcing a pending new admin and the new address should then claim its ownership.        
A similar issue was reported in a previous contest and was assigned a severity of medium: [code-423n4/2021-06-realitycards-findings#105](https://github.com/code-423n4/2021-06-realitycards-findings/issues/105).
````solidity
SafEth.sol
SfrxEth.sol
Reth.sol
WstEth.sol
````

# [N-04] Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`)
While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist.      
     
There is 21 instances of this issue:
````solidity
contracts/SafEth/SafEth.sol
54      minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
55:     maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum

75:     10 ** 18;

80      preDepositPrice = 10 ** 18; // initializes with a price of 1 // @audit n:Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)
81:     else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

95      ) * depositAmount) / 10 ** 18; // @audit n:Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)
96:     totalStakeValueEth += derivativeReceivedEthValue;
````
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L54-L55](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L54-L55)
````solidity
contracts/SafEth/derivatives/Reth.sol
44:     maxSlippage = (1 * 10 ** 16); // 1%

171     uint rethPerEth = (10 ** 36) / poolPrice(); // @audit n:Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)
172
173     uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *// @audit n:Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)
174:    ((10 ** 18 - maxSlippage))) / 10 ** 18);

214     RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18); // @audit n:Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)
215:    else return (poolPrice() * 10 ** 18) / (10 ** 18);
````
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L44](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L44)
````solidity
contracts/SafEth/derivatives/SfrxEth.sol

38:     maxSlippage = (1 * 10 ** 16); // 1%

74      uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *// @audit n:Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)
75:            (10 ** 18 - maxSlippage)) / 10 ** 18;

113     10 ** 18 // @audit n:Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)
114     );
115:    return ((10 ** 18 * frxAmount) /
````
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L38](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L38)
````solidity
contracts/SafEth/derivatives/WstEth.sol
35:     maxSlippage = (1 * 10 ** 16);

60:     uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;

87:     return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
````
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L35](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L35)

# [N-05] `Empty blocks` should be removed or Emit something
## Vulnerability details
Code contains empty blocks.
````solidity
4 results - 4 files

contracts/SafEth/derivatives/WstEth.sol
97:     receive() external payable {}

contracts/SafEth/derivatives/SfrxEth.sol
126:    receive() external payable {}

contracts/SafEth/derivatives/Reth.sol
244:    receive() external payable {}

contracts/SafEth/SafEth.sol
252:    receive() external payable {}
````
## Recommended mitigation steps
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

# [N-05] Add an event for critical parameter changes
Adding events for critical parameter changes will facilitate offchain monitoring and indexing.
- [https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58-L60](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58-L60)
- [https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107-L114](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107-L114)
- [https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L156-L204](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L156-L204)
- [https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53)
- [https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60-L88](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60-L88)
- [https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L94-L106](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L94-L106)
- [https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48-L50](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48-L50)
- [https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56-L67](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56-L67)
- [https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L73-L81](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L73-L81)

# [N-06] Import declarations should import specific identifiers, rather than the whole file
Using import declarations of the form `import {<identifier_name>} from "some/file.sol"` avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation.

There are 31 instances in 4 files of this issue. 

# [N-07] Use of floating pragmas
## Vulnerability details
### Impact
Floating pragmas are used throughout the codebase. Contracts should be deployed with the same compiler version that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated or different compiler version that might introduce bugs that affect the contract system negatively.
### Proof of Concept
[https://swcregistry.io/docs/SWC-103](https://swcregistry.io/docs/SWC-103)      
      
Floating pragmas are used throughout the protocol.
## Tools Used
Manual code review
## Recommended Mitigation Steps
pragma solidity `0.8.13`; should be used in all files instead of pragma solidity `^0.8.13`;
The pragma version should be locked on a specific version while considering know bugs for the chosen compiler version.

# [N-08] Insufficient coverage
## Description
The test coverage rate of the project is 98.17%. Testing all functions is best practice in terms of security criteria.
------------------------------------------------|----------|----------|----------|----------|----------------|
File                                            |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
------------------------------------------------|----------|----------|----------|----------|----------------|
 SafEth\                                        |      100 |       95 |      100 |      100 |                |
  SafEth.sol                                    |      100 |       95 |      100 |      100 |                |
  SafEthStorage.sol                             |      100 |      100 |      100 |      100 |                |
 SafEth\derivatives\                            |    96.34 |    64.29 |    89.29 |     96.3 |                |
  Reth.sol                                      |    97.62 |       75 |    91.67 |    97.56 |             51 |
  SfrxEth.sol                                   |       95 |       50 |     87.5 |       95 |             45 |
  WstEth.sol                                    |       95 |       50 |     87.5 |       95 |             42 |
 interfaces\                                    |      100 |      100 |      100 |      100 |                |
  IDerivative.sol                               |      100 |      100 |      100 |      100 |                |
  IWETH.sol                                     |      100 |      100 |      100 |      100 |                |
 interfaces\curve\                              |      100 |      100 |      100 |      100 |                |
  IAfEthPool.sol                                |      100 |      100 |      100 |      100 |                |
  ICrvEthPool.sol                               |      100 |      100 |      100 |      100 |                |
  ICrvEthPoolLegacy.sol                         |      100 |      100 |      100 |      100 |                |
  IFrxEthEthPool.sol                            |      100 |      100 |      100 |      100 |                |
  IFxsEthPool.sol                               |      100 |      100 |      100 |      100 |                |
  IStEthEthPool.sol                             |      100 |      100 |      100 |      100 |                |
 interfaces\frax\                               |      100 |      100 |      100 |      100 |                |
  IFrxETHMinter.sol                             |      100 |      100 |      100 |      100 |                |
  IsFrxEth.sol                                  |      100 |      100 |      100 |      100 |                |
 interfaces\lido\                               |      100 |      100 |      100 |      100 |                |
  IWStETH.sol                                   |      100 |      100 |      100 |      100 |                |
  IstETH.sol                                    |      100 |      100 |      100 |      100 |                |
 interfaces\rocketpool\                         |      100 |      100 |      100 |      100 |                |
  RocketDAOProtocolSettingsDepositInterface.sol |      100 |      100 |      100 |      100 |                |
  RocketDepositPoolInterface.sol                |      100 |      100 |      100 |      100 |                |
  RocketStorageInterface.sol                    |      100 |      100 |      100 |      100 |                |
  RocketTokenRETHInterface.sol                  |      100 |      100 |      100 |      100 |                |
 interfaces\uniswap\                            |      100 |      100 |      100 |      100 |                |
  ISwapRouter.sol                               |      100 |      100 |      100 |      100 |                |
  IUniswapV3Factory.sol                         |      100 |      100 |      100 |      100 |                |
  IUniswapV3Pool.sol                            |      100 |      100 |      100 |      100 |                |
 interfaces\uniswap\pool\                       |      100 |      100 |      100 |      100 |                |
  IUniswapV3PoolActions.sol                     |      100 |      100 |      100 |      100 |                |
  IUniswapV3PoolDerivedState.sol                |      100 |      100 |      100 |      100 |                |
  IUniswapV3PoolEvents.sol                      |      100 |      100 |      100 |      100 |                |
  IUniswapV3PoolImmutables.sol                  |      100 |      100 |      100 |      100 |                |
  IUniswapV3PoolOwnerActions.sol                |      100 |      100 |      100 |      100 |                |
  IUniswapV3PoolState.sol                       |      100 |      100 |      100 |      100 |                |
 mocks\                                         |      100 |       50 |      100 |      100 |                |
  DerivativeMock.sol                            |      100 |       50 |      100 |      100 |                |
  IDerivativeMock.sol                           |      100 |      100 |      100 |      100 |                |
  SafEthV2Mock.sol                              |      100 |      100 |      100 |      100 |                |
  SafEthV2MockStorage.sol                       |      100 |      100 |      100 |      100 |                |
------------------------------------------------|----------|----------|----------|----------|----------------|
Due to its capacity, test coverage is expected to be 100%.


# [O-01] Function Naming suggestions
Proper use of `_` as a function name prefix and a common pattern is to prefix internal and private function names with `_`.
This pattern is correctly applied in the Party contracts, however there are some inconsistencies in the libraries.       
      
Instances:
````solidity
contracts/SafEth/derivatives/Reth.sol
66:     function rethAddress() private view returns (address) { // @audit o

83:     function swapExactInputSingleHop(// @audit o

120:    function poolCanDeposit(uint256 _amount) private view returns (bool) {// @audit o

228:    function poolPrice() private view returns (uint256) {// @audit o
````

