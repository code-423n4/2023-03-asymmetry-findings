## QA Report

## N-01

Create a single function to get all `rocketPool` contract addresses for code clarity.

## Summary

RocketPool requires us to retrieve active deployed contract addresses from the RocketStorage contract. We can combine all of these retrievals into one function in the Reth.sol contract. This will make the code cleaner and easier to understand.

[Link to the code on Github]( https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L121-L141)

###  Recommended `getAddress()` function

```solidity

  /**
        @notice - Get contract address
        @dev - per RocketPool Docs query addresses each time it is used
        @param _contractName - name of the rocketpool contract
     */

    function getAddress(
        string memory _contractName
    ) private view returns (address) {
        return
            RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
                keccak256(abi.encodePacked("contract.address", _contractName))
            );
    }


```

### `diff` 

```diff
@@ -59,19 +59,6 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
         maxSlippage = _slippage;
     }
 
-    /**
-        @notice - Get rETH address
-        @dev - per RocketPool Docs query addresses each time it is used
-     */
-    function rethAddress() private view returns (address) {
-        return
-            RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
-                keccak256(
-                    abi.encodePacked("contract.address", "rocketTokenRETH")
-                )
-            );
-    }
-
     /**
         @notice - Swap tokens through Uniswap
         @param _tokenIn - token to swap from
@@ -105,7 +92,7 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
         @notice - Convert derivative into ETH
      */
     function withdraw(uint256 amount) external onlyOwner {
-        RocketTokenRETHInterface(rethAddress()).burn(amount);
+        RocketTokenRETHInterface(getAddress("rocketTokenRETH")).burn(amount);
         // solhint-disable-next-line
         (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
             ""

@@ -118,27 +105,14 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
         @param _amount - amount that will be deposited
      */
     function poolCanDeposit(uint256 _amount) private view returns (bool) {
-        address rocketDepositPoolAddress = RocketStorageInterface(
-            ROCKET_STORAGE_ADDRESS
-        ).getAddress(
-                keccak256(
-                    abi.encodePacked("contract.address", "rocketDepositPool")
-                )
-            );
+        address rocketDepositPoolAddress = getAddress("rocketDepositPool");
         RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(
                 rocketDepositPoolAddress
             );
 
-        address rocketProtocolSettingsAddress = RocketStorageInterface(
-            ROCKET_STORAGE_ADDRESS
-        ).getAddress(
-                keccak256(
-                    abi.encodePacked(
-                        "contract.address",
-                        "rocketDAOProtocolSettingsDeposit"
-                    )
-                )
-            );
+        address rocketProtocolSettingsAddress = getAddress(
+            "rocketDAOProtocolSettingsDeposit"
+        );
         RocketDAOProtocolSettingsDepositInterface rocketDAOProtocolSettingsDeposit = RocketDAOProtocolSettingsDepositInterface(
                 rocketProtocolSettingsAddress
             );

@@ -155,14 +129,7 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
      */
     function deposit() external payable onlyOwner returns (uint256) {
         // Per RocketPool Docs query addresses each time it is used
-        address rocketDepositPoolAddress = RocketStorageInterface(
-            ROCKET_STORAGE_ADDRESS
-        ).getAddress(
-                keccak256(
-                    abi.encodePacked("contract.address", "rocketDepositPool")
-                )
-            );
-
+        address rocketDepositPoolAddress = getAddress("rocketDepositPool");
         RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(
                 rocketDepositPoolAddress
             );
@@ -176,7 +143,7 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
             IWETH(W_ETH_ADDRESS).deposit{value: msg.value}();
             uint256 amountSwapped = swapExactInputSingleHop(
                 W_ETH_ADDRESS,
-                rethAddress(),
+                getAddress("rocketTokenRETH"),
                 500,
                 msg.value,
                 minOut
@@ -184,13 +151,7 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
 
             return amountSwapped;
         } else {
-            address rocketTokenRETHAddress = RocketStorageInterface(
-                ROCKET_STORAGE_ADDRESS
-            ).getAddress(
-                    keccak256(
-                        abi.encodePacked("contract.address", "rocketTokenRETH")
-                    )
-                );
+            address rocketTokenRETHAddress = getAddress("rocketTokenRETH");
             RocketTokenRETHInterface rocketTokenRETH = RocketTokenRETHInterface(
                 rocketTokenRETHAddress
             );
@@ -211,7 +172,8 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
     function ethPerDerivative(uint256 _amount) public view returns (uint256) {
         if (poolCanDeposit(_amount))
             return
-                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
+                RocketTokenRETHInterface(getAddress("rocketTokenRETH"))
+                    .getEthValue(10 ** 18);
         else return (poolPrice() * 10 ** 18) / (10 ** 18);
     }

@@ -219,20 +181,14 @@ contract Reth is IDerivative, Initializable, OwnableUpgradeable {
         @notice - Total derivative balance
      */
     function balance() public view returns (uint256) {
-        return IERC20(rethAddress()).balanceOf(address(this));
+        return IERC20(getAddress("rocketTokenRETH")).balanceOf(address(this));
     }
 
     /**
         @notice - Price of derivative in liquidity pool
      */
     function poolPrice() private view returns (uint256) {
-        address rocketTokenRETHAddress = RocketStorageInterface(
-            ROCKET_STORAGE_ADDRESS
-        ).getAddress(
-                keccak256(
-                    abi.encodePacked("contract.address", "rocketTokenRETH")
-                )
-            );
+        address rocketTokenRETHAddress = getAddress("rocketTokenRETH");
         IUniswapV3Factory factory = IUniswapV3Factory(UNI_V3_FACTORY);
         IUniswapV3Pool pool = IUniswapV3Pool(
             factory.getPool(rocketTokenRETHAddress, W_ETH_ADDRESS, 500)

```

