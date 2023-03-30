https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L141

Comment:
/// @audit GAS OPTIMIZATION: should be another statement above this line: IDerivative derivative = derivatives[i]; And then this line should be: if (derivative.balance() != 0)

https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L142

Comment:
/// @audit GAS OPTIMIZATION: this line should be: derivative.withdraw(derivative.balance());

https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L146

Comment:
/// @audit LOW RISK/GAS OPTIMIZATION: here we should add: require(ethAmountToRebalance != 0);

https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L148

Comment:
/// @audit GAS OPTIMIZATION: should be another statement above this line: uint256 weight = weights[i]; And then this line should be: if (weight == 0) continue; Because the ethAmountToRebalance should be checked in require statement above!

https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L149

Comment:
/// @audit GAS OPTIMIZATION: this line should be: uint256 ethAmount = (ethAmountToRebalance * weight) / 

https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L152

Comment:
/// @audit GAS OPTIMIZATION: this line should be: (bool,) = derivative.deposit{value: ethAmount}(); And then below this line, add: require(sent, "Failed to deposit Ether");

https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L172

Comment:
/// @audit GAS OPTIMIZATION: should be another statement above this line: uint256 weight = weights[i]; And then this line should be: localTotalWeight += weight;

https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L192

Comment:
/// @audit GAS OPTIMIZATION: should be another statement above this line: uint256 weight = weights[i]; And then this line should be: localTotalWeight += weight;

https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L233

Comment:
/// @audit GAS OPTIMIZATION: could use 1 and 2 instead of 1 and 0 or true and false. This would save 31 gas per call.

https://github.com/code-423n4/2023-03-asymmetry/blob/a8dd9399565ac608860dcadd7b16ff04aee06cb7/contracts/SafEth/SafEth.sol#L242

Comment:
/// @audit GAS OPTIMIZATION: could use 1 and 2 instead of 1 and 0 or true and false. This would save 31 gas per call.