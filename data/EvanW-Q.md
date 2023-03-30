[QA-01] **NatSpec comments provide rich code documentation. The following functions are some examples that miss the `@param` and/or `@return` comments.**

```solidity
contracts/SafEth/derivativrs/WstEth.sol
48: function setMaxSlippage(uint256 _slippage) external onlyOwner {
56: function withdraw(uint256 _amount) external onlyOwner {
86: function ethPerDerivative(uint256 _amount) public view returns (uint256) {

contracts/SafEth/derivativrs/SfrxEth.sol
51: function setMaxSlippage(uint256 _slippage) external onlyOwner {
111: function ethPerDerivative(uint256 _amount) public view returns (uint256) {

contracts/SafEth/derivativrs/Reth.sol
103: function withdraw(uint256 amount) external onlyOwner {
```