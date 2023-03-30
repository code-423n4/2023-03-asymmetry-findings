## QA REPORT

| |Issue|
|-|:-|
| [01] | LACK OF LIMITS FOR SETTING `maxSlippage` IN `Reth`, `SfrxEth`, AND `WstEth` CONTRACTS |
| [02] | USERS CANNOT SET OWN SLIPPAGE WHEN STAKING AND UNSTAKING |
| [03] | ETH CAN BE SENT TO `SafEth`, `Reth`, `SfrxEth`, AND `WstEth` CONTRACTS ACCIDENTALLY |
| [04] | USERS' ACCIDENTALLY SENT frxETH AND stETH CAN BE LOST |
| [05] | 3RD-PARTY CONTRACT ADDRESSES ARE HARDCODED |
| [06] | MISSING `address(0)` CHECKS FOR CRITICAL ADDRESS INPUTS |
| [07] | VULNERABILITIES IN VERSION 4.8.0 OF `@openzeppelin/contracts` AND VERSION 4.8.1 OF `@openzeppelin/contracts-upgradeable` |
| [08] | SOLIDITY VERSION `0.8.19` CAN BE USED |
| [09] | `rebalanceToWeights` FUNCTION DOES NOT NEED TO LOOP THROUGH DERIVATIVES WHEN `ethAmountToRebalance == 0` IS TRUE |
| [10] | `adjustWeight` FUNCTION DOES NOT NEED TO LOOP THROUGH DERIVATIVES AND INCREASE `localTotalWeight` FOR UPDATING `totalWeight` |
| [11] | `addDerivative` FUNCTION DOES NOT NEED TO LOOP THROUGH DERIVATIVES AND INCREASE `localTotalWeight` FOR UPDATING `totalWeight` |
| [12] | CONSTANTS CAN BE USED INSTEAD OF MAGIC NUMBERS |
| [13] | UNNECESSARY OPERATION CAN BE REMOVED |
| [14] | `uint256` CAN BE USED INSTEAD OF `uint` |
| [15] | SAME INTERFACE CAN BE USED FOR CALLING SAME FUNCTION |
| [16] | COMMENT CAN BE MORE DESCRIPTIVE |
| [17] | TYPO IN COMMENT |
| [18] | REDUNDANT `()` CAN BE REMOVED |
| [19] | UNUSED IMPORTS |
| [20] | FLOATING PRAGMAS |
| [21] | INCOMPLETE NATSPEC COMMENTS |

## [01] LACK OF LIMITS FOR SETTING `maxSlippage` IN `Reth`, `SfrxEth`, AND `WstEth` CONTRACTS
When calling the following `Reth.setMaxSlippage`, `SfrxEth.setMaxSlippage`, and `WstEth.setMaxSlippage` functions, there are no limits for setting `maxSlippage` in the `Reth`, `SfrxEth`, and `WstEth` contracts.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58-L60
```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53
```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48-L50
```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

If `maxSlippage` are set to 10 ** 18, calling the following `Reth.deposit`, `SfrxEth.withdraw`, and `WstEth.withdraw` functions will cause `minOut` to be 0 and result in no slippage control. If `maxSlippage` are set to be more than 10 ** 18, calling the `Reth.deposit`, `SfrxEth.withdraw`, and `WstEth.withdraw` functions can revert because `10 ** 18 - maxSlippage` would underflow.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L156-L204
```solidity
    function deposit() external payable onlyOwner returns (uint256) {
        ...
        if (!poolCanDeposit(msg.value)) {
            ...
            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
                ((10 ** 18 - maxSlippage))) / 10 ** 18);
            ...
            uint256 amountSwapped = swapExactInputSingleHop(
                W_ETH_ADDRESS,
                rethAddress(),
                500,
                msg.value,
                minOut
            );
            ...
        } else {
            ...
        }
    }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60-L88
```solidity
    function withdraw(uint256 _amount) external onlyOwner {
        ...
        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
            (10 ** 18 - maxSlippage)) / 10 ** 18;

        IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).exchange(
            1,
            0,
            frxEthBalance,
            minOut
        );
        ...
    }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56-L67
```solidity
    function withdraw(uint256 _amount) external onlyOwner {
        ...
        uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
        IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);
        ...
    }
```

As a mitigation, to prevent the `Reth.deposit`, `SfrxEth.withdraw`, and `WstEth.withdraw` functions from behaving unexpectedly, the `Reth.setMaxSlippage`, `SfrxEth.setMaxSlippage`, and `WstEth.setMaxSlippage` functions can be updated to only allow `maxSlippage` to be set to values that cannot exceed certain limits, which are reasonable values that are less than 10 ** 18.

## [02] USERS CANNOT SET OWN SLIPPAGE WHEN STAKING AND UNSTAKING
Only the owner of the `SafEth` contract can call the following `SafEth.setMaxSlippage` function to set `maxSlippage` that is used in the corresponding derivative contract. This means that users cannot set their own slippage when staking and unstaking.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202-L208
```solidity
    function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
    ) external onlyOwner {
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
    }
```

When calling the following `SafEth.stake` function, `derivative.deposit{value: ethAmount}()` is called. If the corresponding derivative contract's `deposit` function is like the `Reth.deposit` function that swaps the deposited ETH's corresponding WETH amount for the derivative tokens according to `maxSlippage`, the swapped derivative token amount can be less than expected to the user because she or he cannot specify her or his own `maxSlippage` against the potential price manipulation during the swap. This can then cause a less-than-expected SafEth token amount to be minted to the user.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63-L101
```solidity
    function stake() external payable {
        ...
        uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
        for (uint i = 0; i < derivativeCount; i++) {
            uint256 weight = weights[i];
            IDerivative derivative = derivatives[i];
            if (weight == 0) continue;
            uint256 ethAmount = (msg.value * weight) / totalWeight;

            // This is slightly less than ethAmount because slippage
            uint256 depositAmount = derivative.deposit{value: ethAmount}();
            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
                depositAmount
            ) * depositAmount) / 10 ** 18;
            totalStakeValueEth += derivativeReceivedEthValue;
        }
        // mintAmount represents a percentage of the total assets in the system
        uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
        _mint(msg.sender, mintAmount);
        ...
    }
```

When calling the following `SafEth.unstake` function, `derivatives[i].withdraw(derivativeAmount)` is called. If the corresponding derivative contract's `withdraw` function is like the `SfrxEth.withdraw` or `WstEth.withdraw` function that exchanges the withdrawn derivative token amount for ETH according to `maxSlippage`, the exchanged ETH amount can be less than expected to the user because she or he cannot specify her or his own `maxSlippage` against the potential price manipulation during the exchange. This can then cause a less-than-expected ETH amount to be transferred to the user. Moreover, when the derivative token's market crashes, a user might want to unstake and withdraw ETH as fast as possible to avoid more losses. However, because she or he cannot specify her or his own slippage control, calling the `SafEth.unstake` function, which calls `derivatives[i].withdraw(derivativeAmount)`, can revert due to that `maxSlippage` set by the owner of the `SafEth` contract is too low. In this case, such user cannot unstake and withdraw her or his ETH in a timely manner and loses more ETH as time moves on.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108-L129
```solidity
    function unstake(uint256 _safEthAmount) external {
        ...
        uint256 safEthTotalSupply = totalSupply();
        uint256 ethAmountBefore = address(this).balance;

        for (uint256 i = 0; i < derivativeCount; i++) {
            // withdraw a percentage of each asset based on the amount of safETH
            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
            if (derivativeAmount == 0) continue; // if derivative empty ignore
            derivatives[i].withdraw(derivativeAmount);
        }
        _burn(msg.sender, _safEthAmount);
        uint256 ethAmountAfter = address(this).balance;
        uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;
        // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
            ""
        );
        require(sent, "Failed to send Ether");
        ...
    }
```

As a mitigation, the `SafEth.stake` and `SafEth.unstake` functions can be updated to include an input for specifying `maxSlippage`. The derivative contracts' `deposit` and `withdraw` functions can also be updated to include an input that corresponds to `maxSlippage` from the `SafEth.stake` and `SafEth.unstake` functions. These changes would allow users to set their own slippage when staking and unstaking.

## [03] ETH CAN BE SENT TO `SafEth`, `Reth`, `SfrxEth`, AND `WstEth` CONTRACTS ACCIDENTALLY
The following `receive` functions do not prevent users from accidentally sending ETH to the `SafEth`, `Reth`, `SfrxEth`, and `WstEth` contracts. To prevent users from losing ETH in this way, each of the following `receive` functions can be updated to check if `msg.sender` is the sender contract where the `SafEth`, `Reth`, `SfrxEth`, or `WstEth` contract should receive ETH from; if `msg.sender` is not such sender contract, calling such `receive` function should revert. For example, the `Reth.receive` function can be updated to check if `msg.sender` is the corresponding rETH contract; if not, calling the `Reth.receive` function should revert.

```solidity
contracts\SafEth\SafEth.sol
  246: receive() external payable {}

contracts\SafEth\derivatives\Reth.sol
  244: receive() external payable {}

contracts\SafEth\derivatives\SfrxEth.sol
  126: receive() external payable {}

contracts\SafEth\derivatives\WstEth.sol
  97: receive() external payable {}
```

## [04] USERS' ACCIDENTALLY SENT frxETH AND stETH CAN BE LOST
When the following `SfrxEth.withdraw` function is called, the `SfrxEth` contract's balance of frxETH is exchanged for ETH, which is eventually withdrawn to the user. Yet, it is possible that other users accidentally send frxETH to the `SfrxEth` contract, and such frxETH amount becomes a part of the `SfrxEth` contract's balance of frxETH. Calling the `SfrxEth.withdraw` function will cause these other users to lose such accidentally sent frxETH.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60-L88
```solidity
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
```

Similarly, when the following `WstEth.withdraw` function is called, the `WstEth` contract's balance of stETH is exchanged for ETH, which is eventually withdrawn to the user. However, it is possible that other users accidentally send stETH to the `WstEth` contract, and such stETH amount becomes a part of the `WstEth` contract's balance of stETH. Calling the `WstEth.withdraw` function will cause these other users to lose such accidentally sent stETH.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56-L67
```solidity
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
```

As a mitigation, to follow the similar pattern used in functions like `WstEth.deposit` below, the `SfrxEth.withdraw` function can be updated to keep track of the `SfrxEth` contract's balances of frxETH before and after `IsFrxEth(SFRX_ETH_ADDRESS).redeem(_amount, address(this), address(this)` is called and only exchange the difference between such frxETH balances for ETH; similarly, the `WstEth.withdraw` function can be updated to record the `WstEth` contract's balances of stETH before and after `IWStETH(WST_ETH).unwrap(_amount)` is called and only exchange the difference between such stETH balances for ETH. Moreover, functions, which are only callable by the trusted admin, can be added in the `SfrxEth` and `WstEth` contracts for transferring ERC20 tokens that are accidentally sent by users back to the corresponding users.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L73-L81
```solidity
    function deposit() external payable onlyOwner returns (uint256) {
        uint256 wstEthBalancePre = IWStETH(WST_ETH).balanceOf(address(this));
        // solhint-disable-next-line
        (bool sent, ) = WST_ETH.call{value: msg.value}("");
        require(sent, "Failed to send Ether");
        uint256 wstEthBalancePost = IWStETH(WST_ETH).balanceOf(address(this));
        uint256 wstEthAmount = wstEthBalancePost - wstEthBalancePre;
        return (wstEthAmount);
    }
```

## [05] 3RD-PARTY CONTRACT ADDRESSES ARE HARDCODED
The 3rd-party contract addresses are hardcoded in the `Reth`, `SfrxEth`, and `WstEth` contracts. If the 3rd-party protocols upgrade these contracts to new addresses or if this protocol wants to launch in a new chain where the hardcoded addresses do not correspond to the needed contracts (for instance, 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2 is not the WETH address on Optimism as shown by https://docs.uniswap.org/contracts/v3/reference/deployments), calling functions that rely on these hardcoded addresses can revert or return or use values that are no longer reliable.

As a mitigation, instead of hardcoding these 3rd-party contract addresses in the `Reth`, `SfrxEth`, and `WstEth` contracts, functions that are only callable by the trusted admin can be added for setting and updating these 3rd-party contract addresses.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L20-L27
```solidity
    address public constant ROCKET_STORAGE_ADDRESS =
        0x1d8f8f00cfa6758d7bE78336684788Fb0ee0Fa46;
    address public constant W_ETH_ADDRESS =
        0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;
    address public constant UNISWAP_ROUTER =
        0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45;
    address public constant UNI_V3_FACTORY =
        0x1F98431c8aD98523631AE4a59f267346ea31F984;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L14-L21
```solidity
    address public constant SFRX_ETH_ADDRESS =
        0xac3E018457B222d93114458476f3E3416Abbe38F;
    address public constant FRX_ETH_ADDRESS =
        0x5E8422345238F34275888049021821E8E08CAa1f;
    address public constant FRX_ETH_CRV_POOL_ADDRESS =
        0xa1F8A6807c402E4A15ef4EBa36528A3FED24E577;
    address public constant FRX_ETH_MINTER_ADDRESS =
        0xbAFA44EFE7901E04E39Dad13167D089C559c1138;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L13-L18
```solidity
    address public constant WST_ETH =
        0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0;
    address public constant LIDO_CRV_POOL =
        0xDC24316b9AE028F1497c275EB9192a3Ea0f67022;
    address public constant STETH_TOKEN =
        0xae7ab96520DE3A18E5e111B5EaAb095312D7fE84;
```

## [06] MISSING `address(0)` CHECKS FOR CRITICAL ADDRESS INPUTS
To prevent unintended behaviors, critical address inputs should be checked against `address(0)`. `address(0)` checks are missing for the `_owner` input variables in the following `initialize` functions. Please consider checking them.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L42-L45
```solidity
    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        ...
    }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L36-L39
```solidity
    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        ...
    }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L33-L36
```solidity
    function initialize(address _owner) external initializer {
        _transferOwnership(_owner);
        ...
    }
```

## [07] VULNERABILITIES IN VERSION 4.8.0 OF `@openzeppelin/contracts` AND VERSION 4.8.1 OF `@openzeppelin/contracts-upgradeable`
As shown in the following code in `package.json`, version 4.8.0 of `@openzeppelin/contracts` and version 4.8.1 of `@openzeppelin/contracts-upgradeable` can be used. As described in https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/4.8.0 and https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts-upgradeable/4.8.1, these versions are vulnerable to incorrect calculation for minting NFTs in batches. To reduce the potential attack surface and be more future-proofed, please consider upgrading this package to at least version 4.8.2.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/package.json#L82-L83
```solidity
    "@openzeppelin/contracts": "^4.8.0",
    "@openzeppelin/contracts-upgradeable": "^4.8.1",
```

## [08] SOLIDITY VERSION `0.8.19` CAN BE USED
Using the more updated version of Solidity can enhance security. As described in https://github.com/ethereum/solidity/releases, Version `0.8.19` is the latest version of Solidity, which "contains a fix for a long-standing bug that can result in code that is only used in creation code to also be included in runtime bytecode". To be more secured and more future-proofed, please consider using Version `0.8.19` for the following contracts.

https://github.com/code-423n4/2023-03-polynomial/blob/main/src/VaultToken.sol#L2
```solidity
contracts\SafEth\SafEth.sol
  2: pragma solidity ^0.8.13;

contracts\SafEth\derivatives\Reth.sol
  2: pragma solidity ^0.8.13;

contracts\SafEth\derivatives\SfrxEth.sol
  2: pragma solidity ^0.8.13;

contracts\SafEth\derivatives\WstEth.sol
  2: pragma solidity ^0.8.13;
```

## [09] `rebalanceToWeights` FUNCTION DOES NOT NEED TO LOOP THROUGH DERIVATIVES WHEN `ethAmountToRebalance == 0` IS TRUE
In the following `rebalanceToWeights` function, when `ethAmountToRebalance == 0` is true, none of the derivatives need to be rebalanced. Thus, the following `rebalanceToWeights` function can be refactored to not loop through the derivatives when `ethAmountToRebalance == 0` is true to make the code more efficient. 

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138-L155
```solidity
    function rebalanceToWeights() external onlyOwner {
        uint256 ethAmountBefore = address(this).balance;
        ...
        uint256 ethAmountAfter = address(this).balance;
        uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;

        for (uint i = 0; i < derivativeCount; i++) {
            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                totalWeight;
            // Price will change due to slippage
            derivatives[i].deposit{value: ethAmount}();
        }
        emit Rebalanced();
    }
```

## [10] `adjustWeight` FUNCTION DOES NOT NEED TO LOOP THROUGH DERIVATIVES AND INCREASE `localTotalWeight` FOR UPDATING `totalWeight`
Calling the following `adjustWeight` function can update `totalWeight`. Yet, this function does not need to loop through the derivatives and increase `localTotalWeight` for updating `totalWeight`. To improve efficiency, this function can be updated to decrease `totalWeight` by the old `weights[_derivativeIndex]` and then increase `totalWeight` by the new `_weight`.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175
```solidity
    function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
        weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit WeightChange(_derivativeIndex, _weight);
    }
```

## [11] `addDerivative` FUNCTION DOES NOT NEED TO LOOP THROUGH DERIVATIVES AND INCREASE `localTotalWeight` FOR UPDATING `totalWeight`
When the following `addDerivative` function is called, the derivatives are looped through and `localTotalWeight` is increased to update `totalWeight`. However, instead of looping through the derivatives and increasing `localTotalWeight`, this function can be updated to directly increase `totalWeight` by the new derivative's `_weight` for higher efficiency.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-L195
```solidity
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
        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    }
```

## [12] CONSTANTS CAN BE USED INSTEAD OF MAGIC NUMBERS
( Please note that the following instances are not found in https://gist.github.com/muratkurtulus/c7a89b0ef411b5b96dd8af23ccd95dc4#nc-3-constants-should-be-defined-rather-than-using-magic-numbers. )

To improve readability and maintainability, a constant can be used instead of the magic number. Please consider replacing the magic numbers, such as `10 ** 18`, used in the following code with constants.

```solidity
contracts\SafEth\SafEth.sol
  54: minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum    
  55: maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum    
  75: 10 ** 18;   
  80: preDepositPrice = 10 ** 18; // initializes with a price of 1   
  81: else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;   
  94: ) * depositAmount) / 10 ** 18;   
  98: uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;   

contracts\SafEth\derivatives\Reth.sol
  44: maxSlippage = (1 * 10 ** 16); // 1% 
  171: uint rethPerEth = (10 ** 36) / poolPrice(); 
  173: uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *  
  174: ((10 ** 18 - maxSlippage))) / 10 ** 18);    
  180: 500,    
  214: RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);  
  215: else return (poolPrice() * 10 ** 18) / (10 ** 18);  
  238: factory.getPool(rocketTokenRETHAddress, W_ETH_ADDRESS, 500) 

contracts\SafEth\derivatives\SfrxEth.sol
  38: maxSlippage = (1 * 10 ** 16); // 1% 
  74: uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *  
  75: (10 ** 18 - maxSlippage)) / 10 ** 18;   
  113: 10 ** 18    
  115: return ((10 ** 18 * frxAmount) /    

contracts\SafEth\derivatives\WstEth.sol
  35: maxSlippage = (1 * 10 ** 16); // 1% 
  87: return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18); 
```

## [13] UNNECESSARY OPERATION CAN BE REMOVED
Multiplying `10 ** 18` and then dividing by `10 ** 18` in `(poolPrice() * 10 ** 18) / (10 ** 18)` in the following `ethPerDerivative` function is unnecessary. To make the code more efficient, please consider updating `(poolPrice() * 10 ** 18) / (10 ** 18)` to `poolPrice()` in the following `ethPerDerivative` function.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L211-L216
```solidity
    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        if (poolCanDeposit(_amount))
            ...
        else return (poolPrice() * 10 ** 18) / (10 ** 18);
    }
```

## [14] `uint256` CAN BE USED INSTEAD OF `uint`
Both `uint` and `uint256` are used in the protocol's code. In favor of explicitness, please consider using `uint256` instead of `uint` in the following code.

```solidity
contracts\SafEth\SafEth.sol
  71: for (uint i = 0; i < derivativeCount; i++)    
  84: for (uint i = 0; i < derivativeCount; i++) {    
  140: for (uint i = 0; i < derivativeCount; i++) {    
  147: for (uint i = 0; i < derivativeCount; i++) {    
  203: uint _derivativeIndex,    
  204: uint _slippage
```

## [15] SAME INTERFACE CAN BE USED FOR CALLING SAME FUNCTION
For better code consistency, the same interface can be used for calling the same function.

Both the `RocketTokenRETHInterface` and `IERC20` interfaces are used for calling the `balanceOf` function in the following code. Please consider using only one of these interfaces.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L194-L199
```solidity
            RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(
                rocketTokenRETHAddress
            );
            uint256 rethBalance1 = rocketTokenRETH.balanceOf(address(this));
            ...
            uint256 rethBalance2 = rocketTokenRETH.balanceOf(address(this));
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L221-L223
```solidity
    function balance() public view returns (uint256) {
        return IERC20(rethAddress()).balanceOf(address(this));
    }
```

Both the `IWStETH` and `IERC20` interfaces are used for calling the `balanceOf` function in the following code. Please consider using only one of these interfaces.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L74-L78
```solidity
        uint256 wstEthBalancePre = IWStETH(WST_ETH).balanceOf(address(this));
        ...
        uint256 wstEthBalancePost = IWStETH(WST_ETH).balanceOf(address(this));
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L93-L95
```solidity
    function balance() public view returns (uint256) {
        return IERC20(WST_ETH).balanceOf(address(this));
    }
```

## [16] COMMENT CAN BE MORE DESCRIPTIVE
The following `ethPerDerivative` function returns how much ETH that `10**18` rETH are worth. For a higher clarity, this function's comment can be updated to `Get price of derivative in terms of ETH for 10**18 rETH`.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L206-L216
```solidity
    /**
        @notice - Get price of derivative in terms of ETH
        ...
     */
    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        if (poolCanDeposit(_amount))
            return
                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
        else return (poolPrice() * 10 ** 18) / (10 ** 18);
    }
```

## [17] TYPO IN COMMENT
For better code quality, `room users amount` can be changed to `room for users amount` in the following comment.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L117
```solidity
        @notice - Check whether or not rETH deposit pool has room users amount
```

## [18] REDUNDANT `()` CAN BE REMOVED
To improve code readability and quality, the redundant `()` can be removed.

The redundant `()` around `(10 ** 18 - maxSlippage)` in the following code can be removed.
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173-L174
```solidity
            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
                ((10 ** 18 - maxSlippage))) / 10 ** 18);
```

The redundant `()` around `rethMinted` in the following code can be removed.
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L202
```solidity
            return (rethMinted);
```

The redundant `()` around `wstEthAmount` in the following code can be removed.
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L80
```solidity
        return (wstEthAmount);
```

## [19] UNUSED IMPORTS
The following interfaces are not used in the corresponding contracts. Please consider removing these `import` statements for better readability and maintainability.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L4-L8
```solidity
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../interfaces/IWETH.sol";
import "../interfaces/uniswap/ISwapRouter.sol";
import "../interfaces/lido/IWStETH.sol";
import "../interfaces/lido/IstETH.sol";
```

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L5
```solidity
import "../../interfaces/frax/IsFrxEth.sol";
```

## [20] FLOATING PRAGMAS
It is a best practice to lock pragmas instead of using floating pragmas to ensure that contracts are tested and deployed with the intended compiler version. Accidentally deploying contracts with different compiler versions can lead to unexpected risks and undiscovered bugs. Please consider locking pragmas for the following files.

```solidity
contracts\SafEth\SafEth.sol
  2: pragma solidity ^0.8.13;

contracts\SafEth\derivatives\Reth.sol
  2: pragma solidity ^0.8.13;

contracts\SafEth\derivatives\SfrxEth.sol
  2: pragma solidity ^0.8.13;

contracts\SafEth\derivatives\WstEth.sol
  2: pragma solidity ^0.8.13;
```

## [21] INCOMPLETE NATSPEC COMMENTS
NatSpec comments provide rich code documentation. The following functions miss the `@param` or `@return` comments. Please consider completing the NatSpec comments for these functions.

```solidity
contracts\SafEth\derivatives\Reth.sol
  50: function name() public pure returns (string memory) {   
  66: function rethAddress() private view returns (address) { 
  107: function withdraw(uint256 amount) external onlyOwner {  
  120: function poolCanDeposit(uint256 _amount) private view returns (bool) {  
  156: function deposit() external payable onlyOwner returns (uint256) {   
  211: function ethPerDerivative(uint256 _amount) public view returns (uint256) {  
  221: function balance() public view returns (uint256) {  
  228: function poolPrice() private view returns (uint256) {   

contracts\SafEth\derivatives\SfrxEth.sol
  44: function name() public pure returns (string memory) {   
  51: function setMaxSlippage(uint256 _slippage) external onlyOwner { 
  94: function deposit() external payable onlyOwner returns (uint256) {   
  111: function ethPerDerivative(uint256 _amount) public view returns (uint256) {  
  122: function balance() public view returns (uint256) {  

contracts\SafEth\derivatives\WstEth.sol
  41: function name() public pure returns (string memory) {   
  48: function setMaxSlippage(uint256 _slippage) external onlyOwner { 
  56: function withdraw(uint256 _amount) external onlyOwner { 
  73: function deposit() external payable onlyOwner returns (uint256) { 
  93: function balance() public view returns (uint256) {  
```