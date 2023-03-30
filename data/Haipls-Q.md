## Haven't require for add duplicate Derivative
The situation is aggravated by the fact that it is not possible to remove the derivative, only to exclude it from the calculations
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182

## Using the wrong OwnableUpgradeable initialization flow
*Instances:*
```solidity
contracts\SafEth\SafEth.sol

53:        _transferOwnership(msg.sender);
```

Recommendation:
```solidity
    __Ownable_init()
```


## There are no methods to retrieve useful data for the user
The user can only check in manual mode whether it is profitable for him to `unstake` or not and how much he will receive eth in end

Recommendation:
Add various auxiliary methods that will allow the user to calculate his final rewards, etc.


## Owner can set incorrect settings
There is no validation for `setMinAmount` and `setMaxAmount`, `addDerivative` methods on valid input parameters.

The owner can mistakenly set 'minAmount > maxAmount', `maxAmount == 0` or set zero address to derivative, which will implicitly block the `stake` or `unstake` method

*Instances*
```solidity
contracts\SafEth\SafEth.sol

214:     function setMinAmount(uint256 _minAmount) external onlyOwner {
215:         minAmount = _minAmount;
216:         emit ChangeMinAmount(minAmount);
217:     }

223:     function setMaxAmount(uint256 _maxAmount) external onlyOwner {
224:         maxAmount = _maxAmount;
225:         emit ChangeMaxAmount(maxAmount);
226:     }
```
Recommendation:
Add at least that `_minAmount <= _maxAmount`, `_maxAmount > 0` requirement and check on zero address for `addDerivative`

## Remove unused import
*Instances*
```solidity
contracts\SafEth\SafEth.sol

4: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
5: import "../interfaces/IWETH.sol";
6: import "../interfaces/uniswap/ISwapRouter.sol";
7: import "../interfaces/lido/IWStETH.sol";
8: import "../interfaces/lido/IstETH.sol";

contracts\SafEth\derivatives\Reth.sol

import "../../interfaces/frax/IsFrxEth.sol";
```

## Constants should be defined rather than using magic numbers
*Instances*
```solidity
contracts\SafEth\derivatives\Reth.sol
238:            factory.getPool(rocketTokenRETHAddress, W_ETH_ADDRESS, 500)


177:             uint256 amountSwapped = swapExactInputSingleHop(
178:                 W_ETH_ADDRESS,
179:                 rethAddress(),
180:                 500,
181:                 msg.value,
182:                 minOut
183:             );
```

## Empty/Unused Function Parameters
Empty or unused function parameters should be commented out as a better and declarative way to silence runtime warning messages.

*Instances:*
```solidity
contracts\SafEth\derivatives\SfrxEth.sol
111:     function ethPerDerivative(uint256 _amount) public view returns (uint256)

contracts\SafEth\derivatives\WstEth.sol
86:     function ethPerDerivative(uint256 _amount) public view returns (uint256)
```

### example solution
```solidity
contracts\SafEth\derivatives\SfrxEth.sol
111:     function ethPerDerivative(uint256 /*_amount*/) public view returns (uint256)
```

## Code style violations
### Both `uint256` and `uint` are used.
Which is misleading that uint can have a different value than `uint256`

https://github.com/ethereum/solidity/issues/14026

*Instances:*
```solidity
contracts\SafEth\SafEth.sol

26:     event Staked(address indexed recipient, uint ethIn, uint safEthOut);
27:     event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);
28:     event WeightChange(uint indexed index, uint weight);
29:     event DerivativeAdded(
30:        address indexed contractAddress,
31:        uint weight,
32:        uint index
33:    );

71:        for (uint i = 0; i < derivativeCount; i++)

84:        for (uint i = 0; i < derivativeCount; i++) {

92:            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(

140:        for (uint i = 0; i < derivativeCount; i++) {

147:        for (uint i = 0; i < derivativeCount; i++) {

203:        uint _derivativeIndex,
204:        uint _slippage

contracts\SafEth\derivatives\Reth.sol

171:            uint rethPerEth = (10 ** 36) / poolPrice();
```
### Order of functions & Also name convention for interla/private function
The order of functions does not correspond to any code style. `private/internal` in the middle of the contract, etc

https://docs.soliditylang.org/en/v0.8.19/style-guide.html

```solidity
contracts\SafEth\SafEth.sol

246:    receive() external payable {}

contracts\SafEth\derivatives\Reth.sol

50:     function name() public pure returns (string memory) {

66:    function rethAddress() private view returns (address) {

83:     function swapExactInputSingleHop(

120:     function poolCanDeposit(uint256 _amount) private view returns (bool) {

228:     function poolPrice() private view returns (uint256) {
244:     receive() external payable {}

contracts\SafEth\derivatives\WstEth.sol

41:    function name() public pure returns (string memory) {

97:    receive() external payable {}

contracts\SafEth\derivatives\SfrxEth.sol

44:    function name() public pure returns (string memory) {

126:    receive() external payable {}
```