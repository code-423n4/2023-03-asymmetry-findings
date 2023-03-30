
# Multiplying by $10^{18}$ and then dividing by $10^{18}$ seems unnecessary.
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215

# _disableInitializers() in the constructor 
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L28 
I may be wrong but this code in the constructor makes it impossible to later call initialize (via proxy)!

