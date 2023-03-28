**SafEth.sol**
- L4/5/6/7/8 - Imports are made that are never used, therefore they generate an extra cost of gas and also generate less comprehensibility of the code since they fill the contract with unused code.

- L73/74/75/92/93/94 - Loss of precision due to rounding - Add scalars so roundings are negligible.
Case: (derivatives[i].ethPerDerivative(derivatives[i].balance()) * derivatives[i].balance()) / 10 ** 18;

- L202 - The function setMaxSlippage() is responsible for defining the slippage of a derivative, but it would seem correct to add a require to the first line that checks that the _derivativeIndex is <= the derivativeCount. Because otherwise the slippage of an uncreated derivative would be setting, which later when it is created, would already have a slippege, this can cause confusion, therefore it is best to validate it and reverse it if this happens.

- L170/171/172/173/190/191/192/193 - This code is repeated in two functions, therefore it would be easier to create a private function and have both use it.
uint256 localTotalWeight = 0;
for (uint256 i = 0; i < derivativeCount; i++)
    localTotalWeight += weights[i];
    totalWeight = localTotalWeight;

- L165/182 - There is a mechanism to add derivatives, but there is no specific mechanism to remove a derivative, the most similar thing is to use adjustWeight() and set the weight to 0, but this mechanism does not alter the derivativeCount, therefore, It would be advisable to be able to have one of these mechanisms since derivatives are not necessarily forever.


**contracts/SafEth/derivatives/Reth.sol**
- L5 - Imports are made that are never used, therefore they generate an extra cost of gas and also generate less comprehensibility of the code since they fill the contract with unused code.

- L7/8/9/10 - Multiple interfaces are imported that are not defined with the same standard as the others. Some interfaces are named as INAME and others as NAMEInterface. This should be modified and have the same standard among all.

- L33 - Missing initializer modifier on constructor
OpenZeppelin recommends that the initializer modifier be applied to constructors in order to avoid potential griefs, social engineering, or exploits. Ensure that the modifier is applied to the implementation contract. If the default constructor is currently being used, it should be changed to be an explicit one with the modifier applied.

- L58/59 - Missing event and or timelock for critical parameter change
Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

- L58/66/107/108/156/179/221/244 - Function ordering does not follow the Solidity style guide
According to the Solidity style guide, functions should be laid out in the following order :constructor(), receive(), fallback(), external, public, internal, private, but the cases below do not follow this pattern.

- L173/174/215 - Loss of precision due to rounding - Add scalars so roundings are negligible.
Example Case: ((((rethPerEth * msg.value) / 10 ** 18) * ((10 ** 18 - maxSlippage))) / 10 ** 18);

- L50/221 - The functions name() and balance() are found as public, but they are not used throughout the contract, the correct thing would be for them to be external.


**contracts/SafEth/derivatives/SfrxEth.sol**
- L27 - Missing initializer modifier on constructor
OpenZeppelin recommends that the initializer modifier be applied to constructors in order to avoid potential griefs, social engineering, or exploits. Ensure that the modifier is applied to the implementation contract. If the default constructor is currently being used, it should be changed to be an explicit one with the modifier applied.

- L51 - Missing event and or timelock for critical parameter change
Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

- L51/60/94/122/126 - Function ordering does not follow the Solidity style guide
According to the Solidity style guide, functions should be laid out in the following order :constructor(), receive(), fallback(), external, public, internal, private, but the cases below do not follow this pattern.

- L44/122 - The functions name() and balance() are found as public, but they are not used throughout the contract, the correct thing would be for them to be external.


**contracts/SafEth/derivatives/WstEth.sol**
- L24 - Missing initializer modifier on constructor
OpenZeppelin recommends that the initializer modifier be applied to constructors in order to avoid potential griefs, social engineering, or exploits. Ensure that the modifier is applied to the implementation contract. If the default constructor is currently being used, it should be changed to be an explicit one with the modifier applied.

- L48 - Missing event and or timelock for critical parameter change
Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

- L48/56/73/86/93/97 - Function ordering does not follow the Solidity style guide
According to the Solidity style guide, functions should be laid out in the following order :constructor(), receive(), fallback(), external, public, internal, private, but the cases below do not follow this pattern.

- L41/86/93 - The functions name(), ethPerDerivative() and balance() are found as public, but they are not used throughout the contract, the correct thing would be for them to be external.
