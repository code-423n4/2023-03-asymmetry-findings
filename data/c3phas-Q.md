### Table of Contents
## QA Findings
- [QA Findings](#qa-findings)
- [Upgradeable contract missing a `__gap` storage variable](#upgradeable-contract-missing-a-__gap-storage-variable)
- [Avoid Naming collisions, totalSupply should be renamed](#avoid-naming-collisions-totalsupply-should-be-renamed)
- [Unused local variables should be removed](#unused-local-variables-should-be-removed)
- [Constants should be defined rather than using magic numbers](#constants-should-be-defined-rather-than-using-magic-numbers)
- [Lock pragmas to specific compiler version](#lock-pragmas-to-specific-compiler-version)
- [Natspec is incomplete](#natspec-is-incomplete)
- [Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. `10**18`)](#use-scientific-notation-eg-1e18-rather-than-exponentiation-eg-1018)
- [Lack of event emission on setters](#lack-of-event-emission-on-setters)
- [Lack of consistenty with using uint vs uint256](#lack-of-consistenty-with-using-uint-vs-uint256)
- [Code Structure Deviates From Best-Practice](#code-structure-deviates-from-best-practice)
- [Import declarations should import specific identifiers, rather than the whole file](#import-declarations-should-import-specific-identifiers-rather-than-the-whole-file)


## Upgradeable contract missing a `__gap` storage variable
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L15-L20
```solidity
File: /contracts/SafEth/SafEth.sol
15: contract SafEth is
16:    Initializable,
17:    ERC20Upgradeable,
18:    OwnableUpgradeable,
19:    SafEthStorage
20:{
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L12
```solidity
File: /contracts/SafEth/derivatives/WstEth.sol
12:    contract WstEth is IDerivative, Initializable, OwnableUpgradeable {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L13
```solidity
File: /contracts/SafEth/derivatives/SfrxEth.sol

13:    contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L19
```solidity
File: /contracts/SafEth/derivatives/Reth.sol
19:  contract Reth is IDerivative, Initializable, OwnableUpgradeable {
```

## Avoid Naming collisions, totalSupply should be renamed 
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L63-L101
```solidity
File: /contracts/SafEth/SafEth.sol
63:    function stake() external payable {

77:        uint256 totalSupply = totalSupply();
78:        uint256 preDepositPrice; // Price of safETH in regards to ETH
79:        if (totalSupply == 0)
80:            preDepositPrice = 10 ** 18; // initializes with a price of 1
81:        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply; 
```
The variable name `totalSupply` collides with the function being called `totalSupply()` . We should prepend the local variable with an underscore

```diff
diff --git a/contracts/SafEth/SafEth.sol b/contracts/SafEth/SafEth.sol
index ebadb4b..6276d98 100644
--- a/contracts/SafEth/SafEth.sol
+++ b/contracts/SafEth/SafEth.sol
@@ -74,11 +74,11 @@ contract SafEth is
                     derivatives[i].balance()) /
                 10 ** 18;

-        uint256 totalSupply = totalSupply();
+        uint256 _totalSupply = totalSupply();
         uint256 preDepositPrice; // Price of safETH in regards to ETH
-        if (totalSupply == 0)
+        if (_totalSupply == 0)
             preDepositPrice = 10 ** 18; // initializes with a price of 1
-        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
+        else preDepositPrice = (10 ** 18 * underlyingValue) / _totalSupply;

         uint256 totalStakeValueEth = 0; // total amount of derivatives worth of ETH in system
         for (uint i = 0; i < derivativeCount; i++) {
```

## Unused local variables should be removed

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L111-L117

The variable `_amount` has not been used anywhere on this function.
```solidity
File: /contracts/SafEth/derivatives/SfrxEth.sol

//@audit: _amount not used anywhere
111:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
112:        uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
113:            10 ** 18
114:        );
115:        return ((10 ** 18 * frxAmount) /
116:            IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).price_oracle());
117:    }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L86
```solidity
File: /contracts/SafEth/derivatives/WstEth.sol

//@audit: _amount
86:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```
## Constants should be defined rather than using magic numbers
There are several occurrences of literal values with unexplained meaning .Literal values in the codebase without an explained meaning make the code harder to read, understand and maintain, thus hindering the experience of developers, auditors and external contributors alike.

Developers should define a constant variable for every magic value used , giving it a clear and self-explanatory name. 

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L44
```solidity
File: /contracts/SafEth/derivatives/Reth.sol

//@audit: 500
180:                500,

//@audit: 500
238:            factory.getPool(rocketTokenRETHAddress, W_ETH_ADDRESS, 500)

//@audit: 1e18, 96 * 2
241:        return (sqrtPriceX96 * (uint(sqrtPriceX96)) * (1e18)) >> (96 * 2);
```

## Lock pragmas to specific compiler version
Contracts should be deployed with the same compiler version and flags that they have been tested the most with. Locking the pragma helps ensure that contracts do not accidentally get deployed using, for example, the latest compiler which may have higher risks of undiscovered bugs. Contracts may also be deployed by others and the pragma indicates the compiler version intended by the original authors.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L2
```solidity
File: /contracts/SafEth/derivatives/WstEth.sol
2:pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L2
```solidity
File: /contracts/SafEth/derivatives/SfrxEth.sol
2:pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L2
```solidity
File: /contracts/SafEth/SafEth.sol
2:pragma solidity ^0.8.13;
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L2
```solidity
File: /contracts/SafEth/derivatives/Reth.sol
2:pragma solidity ^0.8.13;
```


## Natspec is incomplete
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L107
```solidity
File: /contracts/SafEth/derivatives/Reth.sol

//@audit: @param amount
107:    function withdraw(uint256 amount) external onlyOwner {
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48
```solidity
File: /contracts/SafEth/derivatives/WstEth.sol

//@audit: @param _slippage
48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {

//@audit: @param _amount
56:    function withdraw(uint256 _amount) external onlyOwner {

//@audit: @param _amount
86:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
```

## Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. `10**18`)
While the compiler knows to optimize away the exponentiation, it's still better coding practice to use idioms that do not require compiler optimization, if they exist

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L35
```solidity
File: /contracts/SafEth/derivatives/WstEth.sol

//@audit: 10 ** 16
35:        maxSlippage = (1 * 10 ** 16); // 1%

//@audit: 10 ** 18
60:        uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;

//@audit: 10 ** 18
87:        return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L44
```solidity
File: /contracts/SafEth/derivatives/Reth.sol

//@audit: 10 ** 16
44:        maxSlippage = (1 * 10 ** 16); // 1%

//@audit: 10 ** 36
171:            uint rethPerEth = (10 ** 36) / poolPrice();

//@audit: 10 ** 18
173:            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *

//@audit: 10 ** 18
174:                ((10 ** 18 - maxSlippage))) / 10 ** 18);

//@audit: 10 ** 18
214:                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);

//@audit: 10 ** 18
215:        else return (poolPrice() * 10 ** 18) / (10 ** 18);
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L38
```solidity
File: /contracts/SafEth/derivatives/SfrxEth.sol

//@audit: 10 ** 16
38:        maxSlippage = (1 * 10 ** 16); // 1%

@audit: 10 ** 18
74:        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *

//@audit: 10 ** 18
75:            (10 ** 18 - maxSlippage)) / 10 ** 18;

//@audit: 10 ** 18
113:            10 ** 18

//@audit: 10 ** 18
115:        return ((10 ** 18 * frxAmount) /
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L54-L55
```solidity
File: /contracts/SafEth/SafEth.sol

//@audit: 10 ** 17
54:        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum

//@audit: 10 ** 18
55:        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum

//@audit: 10 ** 18
75:                10 ** 18;

//@audit: 10 ** 18
80:            preDepositPrice = 10 ** 18; // initializes with a price of 1

//@audit: 10 ** 18
81:        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

//@audit: 10 ** 18
94:            ) * depositAmount) / 10 ** 18;

//@audit: 10 ** 18
98:        uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
```

## Lack of event emission on setters
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48-L50
```solidity
File: /contracts/SafEth/derivatives/WstEth.sol
48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
49:        maxSlippage = _slippage;
50:    }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L58-L60
```solidity
File: /contracts/SafEth/derivatives/Reth.sol
58:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
59:        maxSlippage = _slippage;
60:    }
```

## Lack of consistenty with using uint vs uint256
Variables have been defined with both `uint` and `uint256`. As this two represent the same thing, we should choose one and be consistent with it.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202-L205
```solidity
File: /contracts/SafEth/SafEth.sol

//@audit: uint _derivativeIndex, uint _slippage
202:    function setMaxSlippage(
203:        uint _derivativeIndex,
204:        uint _slippage
205:    ) external onlyOwner {
```
In the above we use `uint`. The following uses `uint256`

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L214
```solidity
File: /contracts/SafEth/SafEth.sol
//@audit: uint256 _minAmount
214:    function setMinAmount(uint256 _minAmount) external onlyOwner {
```

I propose we stick to `uint256` as it aligns with other lower types such as `uint128, uint64...`

## Code Structure Deviates From Best-Practice
The best-practice layout for a contract should follow the following order: state variables, events, modifiers, constructor and functions. Function ordering helps readers identify which functions they can call and find constructor and fallback functions easier. Functions should be grouped according to their visibility and ordered as: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Some constructs deviate from this recommended best-practice: Modifiers are in the middle of contracts. External/public functions are mixed with internal/private ones. Few events are declared in contracts while most others are in corresponding interfaces.

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L97
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#126
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L244
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L246
The above has receive functions at the end of the file rather than after the constructor as per the docs recommendation.


https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L120
In the above we have private functions defined before public ones, according to the docs it is recommended to have all public function declared before private ones

Also, external and public functions are mixed, we should have external function coming before public functions.

 **Recommendation:**
Consider adopting recommended best-practice for code structure and layout.

See: [https://docs.soliditylang.org/en/v0.8.15/style-guide.html#order-of-layout](https://docs.soliditylang.org/en/v0.8.15/style-guide.html#order-of-layout)


## Import declarations should import specific identifiers, rather than the whole file

Using import declarations of the form `import {<identifier_name>} from "some/file.sol"` avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L4-L8
```solidity
File: /contracts/SafEth/derivatives/WstEth.sol
6:  import "../../interfaces/IDerivative.sol";
7:  import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
8:  import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
9:  import "../../interfaces/curve/IStEthEthPool.sol";
10: import "../../interfaces/lido/IWStETH.sol";
```

The above is through out the whole scope
