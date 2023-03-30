# Gas Optimization

## Gas-01 The `onlyOwner` functions can be set as `payable` to save gas

### Context

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L165-L244](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L165-L244)

The `onlyOwner` functions can be set as `payable` to save gas. Also it is the same for `adjustWeight` and `addDerivative` functions. 

```diff
- function rebalanceToWeights() external onlyOwner {
+ function rebalanceToWeights() external onlyOwner payable {

...

function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
+    ) external onlyOwner payable {
-    ) external onlyOwner {

...

function addDerivative(
        address _contractAddress,
        uint256 _weight
-    ) external onlyOwner {
+    ) external onlyOwner payable {

...

function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
-   ) external onlyOwner { 
+    ) external onlyOwner payable { // @audit the previleged owner should be documented for users to understand the risks
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
    }

    /**
        @notice - Sets the minimum amount a user is allowed to stake
        @param _minAmount - amount to set as minimum stake value
    */
-    function setMinAmount(uint256 _minAmount) external onlyOwner {
+    function setMinAmount(uint256 _minAmount) external onlyOwner payable {
        minAmount = _minAmount;
        emit ChangeMinAmount(minAmount);
    }

    /**
        @notice - Owner only function that sets the maximum amount a user is allowed to stake
        @param _maxAmount - amount to set as maximum stake value
    */
-    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
+    function setMaxAmount(uint256 _maxAmount) external onlyOwner payable {
        maxAmount = _maxAmount;
        emit ChangeMaxAmount(maxAmount);
    }

    /**
        @notice - Owner only function that Enables/Disables the stake function
        @param _pause - true disables staking / false enables staking
    */
-    function setPauseStaking(bool _pause) external onlyOwner {
+    function setPauseStaking(bool _pause) external onlyOwner payable {
        pauseStaking = _pause;
        emit StakingPaused(pauseStaking);
    }

    /**
        @notice - Owner only function that enables/disables the unstake function
        @param _pause - true disables unstaking / false enables unstaking
    */
-    function setPauseUnstaking(bool _pause) external onlyOwner payable {
+    function setPauseUnstaking(bool _pause) external onlyOwner payable { // @audit - gas optimization: use uint256 instead of bool to save gas. 
        pauseUnstaking = _pause;
        emit UnstakingPaused(pauseUnstaking);
    }
```

### Proof of Concept

Before: 

```
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  addDerivative          ·      91062  ·     122807  ·        102462  ·           21  ·          -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  adjustWeight             ·      44611  ·      67311  ·         46519  ·           35  ·          -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  rebalanceToWeigh ·     669450  ·     821670  ·        727618  ·            8  ·          -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  setMaxSlippage       ·      47530  ·       69556  ·          58550  ·             6  ·          -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  setMaxAmount       ·                 -  ·                  -  ·          37155  ·              1  ·           -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  setMaxSlippage       ·     47554  ·        69580  ·          58574  ·              6  ·           -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  setMinAmount        ·                 -  ·                  -  ·          37099  ·              1  ·          -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  setPauseStaking     ·                 -  ·                  -  ·          54296  ·              2  ·          -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  setPauseUnstaking ·                -  ·                   -  ·          37243  ·              2  ·          -  │
```

After: 

```
// -- after changed to payable -- 
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  addDerivative          ·      91038  ·     122783  ·        102438  ·           21  ·          -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  adjustWeight           ·      44587  ·      67287  ·         46495  ·           35  ·          -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  rebalanceToWeigh  ·     669426  ·     821646  ·        727594  ·            8  ·          -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  setMaxAmount       ·                 -  ·                  -   ·         37131  ·              1  ·          -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  setMaxSlippage      ·       47530  ·       69556  ·          58550  ·               6  ·          -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  setMinAmount             ·          -  ·          -  ·         37075  ·            1  ·          -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  setPauseStaking     ·                 -  ·                  -  ·          54272  ·              2  ·          -  │
// ·····················|·········································|·······················|·························|····························|·····················|··················
// |  SafEth    ·  setPauseUnstaking ·                -  ·                   -  ·          37219  ·              2  ·          -  │
```

### Recommendation

Add `payable` to the functions. As a result, it saves gas for the owner to call such functions. 

## G-02 `bool` can be set as `uint256` to save gas

### Context

According to the POC comment: “Booleans are more expensive than uint256 or any type that takes up a full word, because each write operation emits an extra SLOAD to first read the slot’s contents, replace the bits taken up by the boolean, and then write back.“

So it is more gas-efficient to use uint256 instead of bool. 

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEthStorage.sol#L16-L17](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEthStorage.sol#L16-L17)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L228-L244](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L228-L244)

### Code Snippet

[https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27)

### Recommendation

Here is an example for changing the contract. 

The codes which supposed to be changed are: 

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEthStorage.sol#L16-L17](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEthStorage.sol#L16-L17)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232-L244](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L232-L244)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L64](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L64)

[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L109](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L109)

```diff
-		bool public pauseStaking; // true if staking is paused
-   bool public pauseUnstaking; // true if unstaking is pause
+   uint256 public pauseUnstaking; // 1: true, 0: false
+   uint256 public pauseUnstaking; 
```

```diff
+   error notProperValue();

+		function setPauseStaking(uint256 _pause) external onlyOwner {
-		function setPauseStaking(bool _pause) external onlyOwner {
+        if (_pause != 1 || _pause != 0) revert notProperValue();
				pauseStaking = _pause;
-        emit StakingPaused(pauseStaking);
+        emit StakingPaused(pauseStaking == true);
    }

    /**
        @notice - Owner only function that enables/disables the unstake function
        @param _pause - true disables unstaking / false enables unstaking
    */
-    function setPauseUnstaking(bool _pause) external onlyOwner {
+    function setPauseUnstaking(uint256 _pause) external onlyOwner {
+        if (_pause != 1 || _pause != 0) revert notProperValue();
        pauseUnstaking = _pause;
-        emit UnstakingPaused(pauseUnstaking);
+        emit UnstakingPaused(pauseUnstaking == true);
    }
```

```diff
function stake() external payable {
-        require(pauseStaking == false, "staking is paused");
+        require(pauseStaking == 0, "staking is paused");

function unstake(uint256 _safEthAmount) external {
-				require(pauseUnstaking == false, "unstaking is paused");
+				require(pauseUnstaking == 0, "unstaking is paused"); 
```

## Gas-03 Cache the length variable before looping

### Context

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L84](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L113](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L147](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L147)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L171](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L171)

[https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L191](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L191)

The gas of reading a state variable is a lot. I noticed in the code snippets below the value `derivativeCount` is not cached before every loop. This can have a big gas decrease if we cache before the loop. 

### Recommendation

Add this line of code before every loop. And change the `derivativeCount` to cached value e.g. `derivativesLength` in the `for` loop line. 

```diff
+       uint256 derivativesLength = derivativeCount;
-       for (uint256 i = 0; i < derivativeCount; i++)
+       for (uint256 i = 0; i < derivativesLength; i++)
```