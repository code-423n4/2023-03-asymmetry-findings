1)  Unused function parameters should be commented out as a better and declarative way to silence runtime warning messages. As an example, the following functions may have these parameters refactored to:


https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/derivatives/SfrxEth.sol#L111

Recommendation :

```
  function ethPerDerivative(uint256 _amount) public view returns (uint256) {
  
  change to
  
  function ethPerDerivative() public view returns (uint256) {
```

same for :

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L86

```
 function ethPerDerivative(uint256 _amount) public view returns (uint256) {

 change to

 function ethPerDerivative() public view returns (uint256) {
```





2) Inconsistent use of uint and uint256 in events

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L207
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L25

```
event has uint256   (uint256 indexed index, uint256 slippage);
function has uint   (uint _derivativeIndex, uint _slippage)
```
Same for events :

    event Staked(address indexed recipient, uint ethIn, uint safEthOut);
    event Unstaked(address indexed recipient, uint ethOut, uint safEthIn);
    event WeightChange(uint indexed index, uint weight);
    event DerivativeAdded(address indexed contractAddress,uint weight,uint index);