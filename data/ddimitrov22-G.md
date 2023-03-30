## Summary

|        | Issues                                                                                                     | Contexts | Estimated Gas Saved |
| ------ | ---------------------------------------------------------------------------------------------------------- | -------- | ------------------- |
| [G-01] | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables                                     | 4       | -                   |
| [G-02] | Empty blocks should be removed or emit something                                                           | 4        | -                   |
| [G-03] | Avoid using state variable in emit (520 gas)                                                                      | 4        | -                   |
| [G-04] | Remove imports that are not used                                                  | 3        | -                   |
| [G-05] | Optimize names to save gas                                                 | -        | -                   |
| [G-06] | Functions guaranteed to revert when called by normal users can be marked payable                                                  | 17        | -                   |


## [G-01] `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables

    4 results - 1 file

    contracts/SafEth/SafEth.sol:
    72:             underlyingValue +=
    95:             totalStakeValueEth += derivativeReceivedEthValue;
    172:             localTotalWeight += weights[i];
    192:             localTotalWeight += weights[i];


## [G-02] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

    4 results - 4 files

    contracts/SafEth/SafEth.sol:
    245  
    246:     receive() external payable {}
    247  }

    contracts/SafEth/derivatives/Reth.sol:
    243  
    244:     receive() external payable {}
    245  }

    contracts/SafEth/derivatives/SfrxEth.sol:
    125  
    126:     receive() external payable {}
    127  }

    contracts/SafEth/derivatives/WstEth.sol:
    96  
    97:     receive() external payable {}
    98  }

## [G-03] Avoid using state variable in emit (130 gas)

Using a state variable in emitting events wastes wastes 130 gas per event.

    emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    emit ChangeMinAmount(minAmount);
    emit ChangeMaxAmount(maxAmount);
    emit UnstakingPaused(pauseUnstaking);

## [G-04] Remove imports that are not used

The following interfaces are not used:

    import "../interfaces/lido/IWStETH.sol";
    import "../interfaces/lido/IstETH.sol";
    import "../interfaces/uniswap/ISwapRouter.sol";


Consider removing them.

## [G-05] Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

## [G-06] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

    17 results - 4 files

    contracts/SafEth/SafEth.sol:
    138:     function rebalanceToWeights() external onlyOwner {
    168:     ) external onlyOwner {
    185:     ) external onlyOwner {
    205:     ) external onlyOwner {
    214:     function setMinAmount(uint256 _minAmount) external onlyOwner {
    223:     function setMaxAmount(uint256 _maxAmount) external onlyOwner {
    232:     function setPauseStaking(bool _pause) external onlyOwner {
    241:     function setPauseUnstaking(bool _pause) external onlyOwner {

    contracts/SafEth/derivatives/Reth.sol:
    58:     function setMaxSlippage(uint256 _slippage) external onlyOwner {
    107:     function withdraw(uint256 amount) external onlyOwner {
    156:     function deposit() external payable onlyOwner returns (uint256) {

    contracts/SafEth/derivatives/SfrxEth.sol:
    51:     function setMaxSlippage(uint256 _slippage) external onlyOwner {
    60:     function withdraw(uint256 _amount) external onlyOwner {
    94:     function deposit() external payable onlyOwner returns (uint256) {

    contracts/SafEth/derivatives/WstEth.sol:
    48:     function setMaxSlippage(uint256 _slippage) external onlyOwner {
    56:     function withdraw(uint256 _amount) external onlyOwner {
    73:     function deposit() external payable onlyOwner returns (uint256) {