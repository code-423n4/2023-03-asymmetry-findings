| SN | Description |
| --- | --- |
| L-01 | Failure to add a storage gap variable in upgradable contracts |
| L-02 | Unused function named parameter |
| L-03 | Inconsistent `uint256` variables declaration |
| L-04 | Unnecessary code run when `derivativeCount` is zero |
| L-05 | Using floating pragma version |
| L-06 | Use scientific notation in lieu of exponentiation |
| L-07 | The private and internal function names don’t start with an underscore |
| L-08 | Incorrect NatSpec |
| L-09 | Missing input param validation |

## L-01. Failure to add a storage gap variable in upgradable contracts

### Context

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol)

### Impact

Potentially, any Ethereum developer can inherit their contracts from any of the Asymmetry project contracts. However, it’s worth noting, that if a contract having state variables is upgradable, and it has a child contract which declares its own state variables, in case the base contract is modified and new storage variables are added, the newly created storage slots will clash with the child’s ones.

### Proof of Concept

**`BaseContract` version 1:**

```solidity
contract BaseContract is OwnableUpgradeable {
	uint256 baseVar1;
}

contract ChildContract is BaseContract {
	uint256 childVar1;
}
```

**`BaseContract` version 2:**

```solidity
contract BaseContract is OwnableUpgradeable {
	uint256 baseVar1;
	uint256 baseVar2;
}

contract ChildContract is BaseContract {
	uint256 childVar1;
}
```

After adding `baseVar2` state variable in `BaseContract` version 2, the `baseVar2` is assigned the slot previously occupied by `childVar1` state variable in version 1 setup. 

### Recommended Mitigation Steps

Insert a storage gap in the contracts in scope as recommended by OpenZeppelin:

[https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#storage-gaps](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#storage-gaps)

## L-02. Unused function named parameter

### Context

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L111-L117](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L111-L117)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86-L88](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86-L88)

### Vulnerability Details

Unused function named parameter can be misleading.

### Proof of Concept

```solidity
// SfrxEth.sol

function ethPerDerivative(uint256 _amount) public view returns (uint256) {
    uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
        10 ** 18
    );
    return ((10 ** 18 * frxAmount) /
        IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
}
```

```solidity
// WstEth.sol

function ethPerDerivative(uint256 _amount) public view returns (uint256) {
    return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
}
```

As can be seen above, the function param `_amount` has not been used in the function body. 

### Recommended Mitigation Steps

Remove the param name:

```diff
// SfrxEth.sol

- function ethPerDerivative(uint256 _amount) public view returns (uint256) {
+ function ethPerDerivative(uint256) public view returns (uint256) {
    uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
        10 ** 18
    );
    return ((10 ** 18 * frxAmount) /
        IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
}
```

```diff
// WstEth.sol

- function ethPerDerivative(uint256 _amount) public view returns (uint256) {
+ function ethPerDerivative(uint256) public view returns (uint256) {
    return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
}
```

## L-03. Inconsistent `uint256` variables declaration

### Vulnerability Details

The variables of type `uint256` are declared using inconsistently both `uint` and `uint256` notations, which represent the same type.

### Context and Proof of Concept

**`SafEth.sol`**

**`uint256` notation:**

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L21](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L21)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L22](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L22)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L25](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L25)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L25](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L25)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L68](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L68)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L77](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L77)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L78](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L78)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L83](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L83)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L85](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L85)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L88](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L88)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L91](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L91)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L98](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L98)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L111](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L111)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L113)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L115](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L115)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L121](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L121)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L122](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L122)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L139](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L139)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L144](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L144)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L145](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L145)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L149](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L149)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L166](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L166)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L167](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L167)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L170](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L170)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L184](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L184)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L190](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L190)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223)

**`uint` notation:**

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L26](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L26)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L27](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L27)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L28](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L28)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L31](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L31)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L32](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L32)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L71)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L92](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L92)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L140)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L147)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L203](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L203)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L204](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L204)

**`Reth.sol`**

**`uint256` notation:**

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L29](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L29)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L87](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L87)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L88](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L88)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L89](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L89)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L120](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L120)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L156](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L156)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L177](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L177)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L197](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L197)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L199](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L199)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L201](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L201)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L211](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L211)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L221](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L221)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L228](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L228)

**`uint` notation:**

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L228](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L228)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L241](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L241)

## L-04. Unnecessary code run when `derivativeCount` is zero

### Context

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63-L101](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63-L101)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108-L129](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L108-L129)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138-L155](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138-L155)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138-L155](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138-L155)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-L195](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182-L195)

### Vulnerability Details

If any of the following `SafEth` contract functions: `stake`, `unstake`, `rebalanceToWeights`, `adjustWeight`, `addDerivative` is called prior to adding a derivative (via `addDerivative`) and, accordingly, `derivativeCount` is still zero, the respective function will run basically in vain, just spending gas unnecessarily.

### Proof of Concept

As can be seen below, if `derivativeCount` equal zero, no `for` loop will run. Accordingly, all the variables used at the bottom of each function, which where supposed to get meaningful values inside the loop, will remain in their zero-state.

```solidity
function stake() external payable {
    require(pauseStaking == false, "staking is paused");
    require(msg.value >= minAmount, "amount too low");
    require(msg.value <= maxAmount, "amount too high");

    uint256 underlyingValue = 0;

    // Getting underlying value in terms of ETH for each derivative
    for (uint i = 0; i < derivativeCount; i++)
        underlyingValue +=
            (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
                derivatives[i].balance()) /
            10 ** 18;

    uint256 totalSupply = totalSupply();
    uint256 preDepositPrice; // Price of safETH in regards to ETH
    if (totalSupply == 0)
        preDepositPrice = 10 ** 18; // initializes with a price of 1
    else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

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
    emit Staked(msg.sender, msg.value, mintAmount);
}

/**
    @notice - Unstake your safETH into ETH
    @dev - unstakes a percentage of safEth based on its total value
    @param _safEthAmount - amount of safETH to unstake into ETH
*/
function unstake(uint256 _safEthAmount) external {
    require(pauseUnstaking == false, "unstaking is paused");
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
    emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
}
```

```solidity
/**
    @notice - Rebalance each derivative to resemble the weight set for it
    @dev - Withdraws all derivative and re-deposit them to have the correct weights
    @dev - Depending on the balance of the derivative this could cause bad slippage
    @dev - If weights are updated then it will slowly change over time to the correct weight distribution
    @dev - Probably not going to be used often, if at all
*/
function rebalanceToWeights() external onlyOwner {
    uint256 ethAmountBefore = address(this).balance;
    for (uint i = 0; i < derivativeCount; i++) {
        if (derivatives[i].balance() > 0)
            derivatives[i].withdraw(derivatives[i].balance());
    }
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

/**
    @notice - Adds new derivative to the index fund
    @dev - Weights are only in regards to each other, total weight changes with this function
    @dev - If you want exact weights either do the math off chain or reset all existing derivates to the weights you want
    @dev - Weights are approximate as it will slowly change as people stake
    @param _derivativeIndex - index of the derivative you want to update the weight
    @param _weight - new weight for this derivative.
*/
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

/**
    @notice - Adds new derivative to the index fund
    @param _contractAddress - Address of the derivative contract launched by AF
    @param _weight - new weight for this derivative. 
*/
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

### Recommended Mitigation Steps

Add a `require` statement in all of the above functions:

```solidity
require(derivativeCount > 0, "At least one derivative must be added");
```

## L-05. Using floating pragma version

### Vulnerability Details

Floating pragmas are used in all the contracts. However, the contracts should be deployed with the same compiler version. Locking the pragma helps ensuring that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively. 

Reference: [https://swcregistry.io/docs/SWC-103](https://swcregistry.io/docs/SWC-103)

### **Recommended Mitigation Steps**

Lock the pragma version: delete `pragma solidity ^0.8.13` in favor of `pragma solidity 0.8.13` or a higher version.

## L-06. Use scientific notation in lieu of exponentiation

### Context and Recommended Mitigation Steps

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol)

```diff
// SafEth.sol

...

- minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
+ minAmount = 5 * 1e17; // initializing with .5 ETH as minimum
- maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
+ maxAmount = 200 * 1e18; // initializing with 200 ETH as maximum

...

- for (uint i = 0; i < derivativeCount; i++)
-     underlyingValue +=
-         (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
-             derivatives[i].balance()) /
-         10 ** 18;
+ for (uint i = 0; i < derivativeCount; i++)
+     underlyingValue +=
+         (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
+             derivatives[i].balance()) /
+         1e18;

...

- if (totalSupply == 0)
-         preDepositPrice = 10 ** 18; // initializes with a price of 1
-     else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
+ if (totalSupply == 0)
+         preDepositPrice = 1e18; // initializes with a price of 1
+     else preDepositPrice = (1e18 * underlyingValue) / totalSupply;

...

- uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
-         depositAmount
-     ) * depositAmount) / 10 ** 18;
+ uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
+         depositAmount
+     ) * depositAmount) / 1e18;

...

- uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
+ uint256 mintAmount = (totalStakeValueEth * 1e18) / preDepositPrice;

...
```

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol)

```diff
// Reth.sol

...

- maxSlippage = (1 * 10 ** 16); // 1%
+ maxSlippage = (1 * 1e16); // 1%

...

- uint rethPerEth = (10 ** 36) / poolPrice();

-     uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
-         ((10 ** 18 - maxSlippage))) / 10 ** 18);
+ uint rethPerEth = (1e36) / poolPrice();

+     uint256 minOut = ((((rethPerEth * msg.value) / 1e18) *
+         ((1e18 - maxSlippage))) / 1e18);

...

- if (poolCanDeposit(_amount))
-         return
-             RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
-     else return (poolPrice() * 10 ** 18) / (10 ** 18);
+ if (poolCanDeposit(_amount))
+         return
+             RocketTokenRETHInterface(rethAddress()).getEthValue(1e18);
+     else return (poolPrice() * 1e18) / (1e18);

...
```

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol)

```diff
// SfrxEth.sol

...

- maxSlippage = (1 * 10 ** 16); // 1%
+ maxSlippage = (1 * 1e16); // 1%

...

- uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
-     (10 ** 18 - maxSlippage)) / 10 ** 18;
+ uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 1e18) *
+     (1e18 - maxSlippage)) / 1e18;

...

- function ethPerDerivative(uint256 _amount) public view returns (uint256) {
-     uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
-         10 ** 18
-     );
-     return ((10 ** 18 * frxAmount) /
-         IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
- }
+ function ethPerDerivative(uint256 _amount) public view returns (uint256) {
+     uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
+         1e18
+     );
+     return ((1e18 * frxAmount) /
+         IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
+ }

...
```

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol)

```diff
// WstEth.sol

...

- function initialize(address _owner) external initializer {
-     _transferOwnership(_owner);
-     maxSlippage = (1 * 10 ** 16); // 1%
- }
+ function initialize(address _owner) external initializer {
+     _transferOwnership(_owner);
+     maxSlippage = (1 * 1e16); // 1%
+ }

...

- uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
+ uint256 minOut = (stEthBal * (1e18 - maxSlippage)) / 1e18;

...

- function ethPerDerivative(uint256 _amount) public view returns (uint256) {
-     return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
- }
+ function ethPerDerivative(uint256 _amount) public view returns (uint256) {
+     return IWStETH(WST_ETH).getStETHByWstETH(1e18);
+ }

...
```

## L-07. The `private` and `internal` function names don’t start with an underscore

### Vulnerability Details

Even though it is not part of the Solidity Style Guide, nowadays, it is common practice (mainly promoted by OpenZeppelin) to prepend an underscore to the `private` and `internal` function names. In fact, it is not only a matter of being in vogue, this convention is very useful during smart contracts development.

### Context and Recommended Mitigation Steps

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L66](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L66)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L83-L89](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L83-L89)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L120](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L120)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L228](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L228)

```diff
// Reth.sol

...

- function rethAddress() private view returns (address) {
+ function _rethAddress() private view returns (address) {

...

- function swapExactInputSingleHop(
-         address _tokenIn,
-         address _tokenOut,
-         uint24 _poolFee,
-         uint256 _amountIn,
-         uint256 _minOut
-     ) private returns (uint256 amountOut) {
+ function _swapExactInputSingleHop(
+         address _tokenIn,
+         address _tokenOut,
+         uint24 _poolFee,
+         uint256 _amountIn,
+         uint256 _minOut
+     ) private returns (uint256 amountOut) {

...

- function poolCanDeposit(uint256 _amount) private view returns (bool) {
+ function _poolCanDeposit(uint256 _amount) private view returns (bool) {

...

- function poolPrice() private view returns (uint256)
+ function _poolPrice() private view returns (uint256)
```

## L-08. Incorrect NatSpec

### Context

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L157-L175](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L157-L175)

```solidity
/**
    @notice - Adds new derivative to the index fund
    @dev - Weights are only in regards to each other, total weight changes with this function
    @dev - If you want exact weights either do the math off chain or reset all existing derivates to the weights you want
    @dev - Weights are approximate as it will slowly change as people stake
    @param _derivativeIndex - index of the derivative you want to update the weight
    @param _weight - new weight for this derivative.
*/
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

### Recommended Mitigation Steps

Perform the following correction:

```diff
- @notice - Adds new derivative to the index fund
+ @notice - Updates the weight of a certain derivative and adjusts the total weight of derivatives
```

## L-09. Missing input param validation

### Context

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175)

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

### Impact

Failure to validate `_derivativeIndex` input param can result in `adjustWeight` function not performing as expected, i.e. updating correctly the weight of a certain derivative represented by its index, `_derivativeIndex`, as well as updating `totalWeight`.

### Proof of Concept

Let’s assume that currently there are 3 derivatives (`derivativeCount` equals `3`), which makes sense, as this is the initial number of the derivatives expected to be deployed at the time of this audit. Accordingly, the index numbers of these derivatives will be `0`, `1`, and `2` . Now, in case the `owner` calls the `adjustWeight` function passing erroneously `5` as `_derivativeIndex`, the weight of a non-existent derivative will be set, and the `totalWeight` will not get updated.

### Recommended Mitigation Steps

Add the following `require` statement:

```solidity
require(_derivativeIndex <= derivativeCount, "Non-existent derivative index");
```