# [G-01] Setting the constructor to `payable`
You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor `payable`. Making the constructor payable eliminates the need for an initial check of `msg.value == 0` and saves 13 gas on deployment with no security risks.
## Context
4 results - 4 files
````solidity
contracts/SafEth/derivatives/WstEth.sol
24:     constructor() {
````
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L24](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L24)
````solidity
contracts/SafEth/derivatives/Reth.sol
33:     constructor() {
````
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L33](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L33)
````solidity
contracts/SafEth/derivatives/SfrxEth.sol
27:     constructor() {
````
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L27](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L27)
````solidity
contracts/SafEth/derivatives/WstEth.sol
24:     constructor() {
````
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L24](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L24)
## Recommended mitigation steps
Set the constructor to `payable`.

# [G-02] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables (`-=` too)
Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8). Subtractions act the same way.       
        
There are 10 instances of this issue.

# [G-03] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are `CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.       
    
There are 14 instances of this issue:
````solidity
contracts/SafEth/SafEth.sol
138:       function rebalanceToWeights() external onlyOwner {

167        function adjustWeight(
168                uint256 _derivativeIndex,
169                uint256 _weight
170:            ) external onlyOwner {

185        function addDerivative(
186                address _contractAddress,
187                uint256 _weight
188:            ) external onlyOwner {

206        function setMaxSlippage(
207                uint _derivativeIndex,
208                uint _slippage
209:            ) external onlyOwner {

218:       function setMinAmount(uint256 _minAmount) external onlyOwner {

227:       function setMaxAmount(uint256 _maxAmount) external onlyOwner {

236:       function setPauseStaking(bool _pause) external onlyOwner {

245:       function setPauseUnstaking(bool _pause) external onlyOwner {
````
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138)
````solidity
contracts/SafEth/derivatives/Reth.sol
58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

107:    function withdraw(uint256 amount) external onlyOwner {
````
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58)
````solidity
contracts/SafEth/derivatives/SfrxEth.sol
51:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

60:     function withdraw(uint256 _amount) external onlyOwner {
````
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51)
````solidity
contracts/SafEth/derivatives/WstEth.sol
48:     function setMaxSlippage(uint256 _slippage) external onlyOwner {

56:     function withdraw(uint256 _amount) external onlyOwner {
````
[https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48)

# [G-04] Using Solidity version `0.8.17` will provide an overall gas optimization
Using at least `0.8.10` will save gas due to skipped `extcodesize` check if there is a return value. Currently the contracts are compiled using version ^0.8.13. It is easily changeable to `0.8.17` using the command `sed -i 's/0\.8\.7/^0.8.0/' test/*.sol && sed -i '4isolc = "0.8.17"' foundry.toml`.

# [G-05] Use a more recent version of solidity
In `0.8.15` the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In `0.8.17` prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.
````solidity
contracts/SafEth/SafEth.sol
2:      pragma solidity ^0.8.13;

contracts/SafEth/derivatives/Reth.sol
2:      pragma solidity ^0.8.13;

contracts/SafEth/derivatives/SfrxEth.sol
2:      pragma solidity ^0.8.13;

contracts/SafEth/derivatives/WstEth.sol
2:      pragma solidity ^0.8.13;
````