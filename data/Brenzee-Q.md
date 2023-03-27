# QA Report

## [L-01] `receive()` function allows anyone to send ETH
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L246
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L126
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L97
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L244

Intentional use of `receive()` in `SafEth.sol` is to receive ETH when `derivative.withdraw` is called. Since there are no checks on `msg.sender`, anyone can send ETH and it will result in a loss of funds. Suggest checking if the `msg.sender` is one of the derivative contracts.

The same goes for derivative contracts - ETH is only expected to be received from known protocol contracts. And because all of the derivative contracts transfer ETH via `withdraw` function where the whole ETH balance is sent, these funds can be stolen.

```solidity
 receive() external payable {}
```

### Example
Alice accidentally sends ETH straight to `WstEthStaking` contract instead of calling `safEthProxy.stake` .

Bob sees that Alice has sent ETH to `WstEthStaking` contract. He stakes and unstakes ETH and receives that 1 ETH that has been sent by Alice.

```typescript   
it("Bob should get Alice's sent ETH ", async () => {
      const [_, alice, bob] = await ethers.getSigners();

      // derivatives addresses
      const WstEthStakingAddress = await safEthProxy.derivatives(2);

      const bobBalanceBefore = await ethers.provider.getBalance(bob.address);

      // Transfer ETH to WstEthStakingAddress
      await alice.sendTransaction({
        to: WstEthStakingAddress,
        value: ethers.utils.parseEther("1"),
      });

      // Bob stakes and unstakes 1 ETH
      await safEthProxy.connect(bob).stake({
        value: ethers.utils.parseEther("1"),
      });
      await safEthProxy
        .connect(bob)
        .unstake(await safEthProxy.balanceOf(bob.address));

      const bobBalanceAfter = await ethers.provider.getBalance(bob.address);

      console.log(
        "Bob got additional",
        Number(bobBalanceAfter.sub(bobBalanceBefore).toString()) / 1e18,
        "ETH"
      ); // Bob got additional 0.9914137732229785 ETH (after gas fees)

      // We expect bob to have more ETH than before
      expect(bobBalanceAfter).to.be.gt(bobBalanceBefore);
    });
```

## [L-02] No need to check balance in `deposit()` and `withdraw()` functions in `SfrxEth.sol`

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L98-L104
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L61-L68
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L57-L58

`SfrxEth.sol` - `frxETHMinterContract.submitAndDeposit` and `IsFrxEth(SFRX_ETH_ADDRESS).redeem` returns how many tokens have been received, there is no need to call `balanceOf`. After changing the code to the example below, all tests still succeed.

```diff
@@ -58,14 +58,11 @@ contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {
         @param _amount - Amount to withdraw
      */
     function withdraw(uint256 _amount) external onlyOwner {
-        IsFrxEth(SFRX_ETH_ADDRESS).redeem(
+        uint256 frxEthBalance = IsFrxEth(SFRX_ETH_ADDRESS).redeem(
             _amount,
             address(this),
             address(this)
         );
-        uint256 frxEthBalance = IERC20(FRX_ETH_ADDRESS).balanceOf(
-            address(this)
-        );
         IsFrxEth(FRX_ETH_ADDRESS).approve(
             FRX_ETH_CRV_POOL_ADDRESS,
             frxEthBalance
@@ -95,14 +92,8 @@ contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {
         IFrxETHMinter frxETHMinterContract = IFrxETHMinter(
             FRX_ETH_MINTER_ADDRESS
         );
-        uint256 sfrxBalancePre = IERC20(SFRX_ETH_ADDRESS).balanceOf(
-            address(this)
-        );
-        frxETHMinterContract.submitAndDeposit{value: msg.value}(address(this));
-        uint256 sfrxBalancePost = IERC20(SFRX_ETH_ADDRESS).balanceOf(
-            address(this)
-        );
-        return sfrxBalancePost - sfrxBalancePre;
+        uint256 sfrxReceived = frxETHMinterContract.submitAndDeposit{value: msg.value}(address(this));
+        return sfrxReceived;
     }
```
`WstEth.sol` - `IWStETH(WST_ETH).unwrap(_amount)` function returns how much tokens have been received, no need to call `balanceOf`.
```diff
@@ -54,8 +54,7 @@ contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
         @dev - Owner is set to SafEth contract
      */
     function withdraw(uint256 _amount) external onlyOwner {
-        IWStETH(WST_ETH).unwrap(_amount);
-        uint256 stEthBal = IERC20(STETH_TOKEN).balanceOf(address(this));
+        uint256 stEthBal = IWStETH(WST_ETH).unwrap(_amount);
         IERC20(STETH_TOKEN).approve(LIDO_CRV_POOL, stEthBal);
         uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
         IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);
```

## [L-03] `setMaxSlippage` doesn't validate the input value
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58-L60
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51-L53
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48-L50

The owner can set slippage to 0 or a very big number (even above 100%), which would make `withdraw` functions in derivative contracts always fail. Additionally, if the slippage is too big and under 100%, transaction might get front ran.

```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```

If the slippage is set too high, error `Arithmetic over/underflow` will be thrown.
If the slippage is set to 0 (or very low value), slippage will not be enough and will make the calls fail.

Suggesting to add validation for  `_slippage` value in `setMaxSlippage`, so it is not too high and not 0. Suggest to change type `uint256` to a smaller value type.

## [L-04] `minOut` value has precision loss in `Reth.sol`
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173-L174

`minOut` value has precision loss in `Reth.sol`, which could lead to an incorrectly calculated value.
```solidity
         uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
             ((10 ** 18 - maxSlippage))) / 10 ** 18);
```

Suggesting to change it to:
```solidity
         uint256 minOut = ((rethPerEth * msg.value) *
             (10 ** 18 - maxSlippage)) / 10 ** 36;
```

## [L-05] Too low or too high weight will make `stake()` function unusable

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L175
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84-L96

If the `weight` of the derivative is too low or too high, it may cause `stake()` function to fail most of the time.

SafEth.sol contract code, where weight is changed and used
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

```solidity
    function stake() external payable {
        ...
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
		...
    }
```
**Example 1:**
Admin calls `adjustWeight` where admin mistakenly changes weight to "1" for one of the derivatives.
After that User1 tries to stake, but User1's `stake()` fails. 

**Example 2:**
Admin calls `adjustWeight` where admin mistakenly changes weight to a really big number.
User1 tries to stake and transaction fails.

Suggesting to validate  `weight` in `adjustWeight` and `addDerivative` functions so it is not too high or too low. Additionally suggesting to change  `uint256` to a smaller value type.

## [N-01] Unused function parameter `_amount` 
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L111
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86

`_amount` variable is not used in the functions. Consider commenting out or removing  `amount_` .

SfrxEth.sol
```solidity
    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
            10 ** 18
        );
        return ((10 ** 18 * frxAmount) /
            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
    }
```

WstEth.sol
```solidity
    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
    }
```

## [N-02] No need to multiply and divide with the same number
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215

Multiplying and dividing with 10 ** 18 will not change the result, suggest changing it to just `poolPrice()`

```solidity
    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        if (poolCanDeposit(_amount))
            return
                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
        else return (poolPrice() * 10 ** 18) / (10 ** 18);
    }
```


## [N-03] Incorrect description in the comment
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L157-L164

@notice description on `adjustWeight` function is incorrect. 

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
  ) external onlyOwner
```

