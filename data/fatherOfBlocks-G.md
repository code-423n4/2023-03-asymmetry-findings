**SafEth.sol**
- L75/80/81/94/98 - The number 10 ** 18 is used multiple times, it would be advisable to create a variable in memory with this value so as not to perform the same operation multiple times.

- L88/91/144/145/149/152 - A variable is created but it is only used once, therefore it does not make much sense to create a variable, generating more gas costs.


**contracts/SafEth/derivatives/Reth.sol**
- L201 - The operation rethBalance2 - rethBalance1 is performed and in L200 it is validated that rethBalance2 > rethBalance1, therefore the subtraction can be performed unchecked and less gas expense would be generated.

- L201/202 - A variable is created but it is only used once, therefore it does not make much sense to create a variable, generating more gas costs.


**contracts/SafEth/derivatives/SfrxEth.sol**
- L113/115 - The 10 ** 18 operation is used multiple times, a variable could be created directly in memory and avoid doing the operation twice.


**contracts/SafEth/derivatives/WstEth.sol**
- L79/80 - A variable is created but it is only used once, therefore it does not make much sense to create a variable, generating more gas costs.

- L60 - The 10 ** 18 operation is used multiple times, a variable could be created directly in memory and avoid doing the operation twice.
