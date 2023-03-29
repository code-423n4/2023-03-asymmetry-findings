https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L88

a minor issue related to the rounding down behavior of Solidity. 
Specifically, due to this behavior, there may be a small amount of leftover Ether (dust) in the contract after a transaction.

solutions:

1. Modify the contract code to subtract the total ethAmount used in each iteration from the msg.value in the final iteration. This will ensure that the exact amount of Ether is used in the contract, leaving no dust.

2. Alternatively, add a function to the contract that allows the contract owner to withdraw any remaining dust Ether from the contract.

3. transfer dust to msg.sender at the end of the function (not recommend: gas might be more expensive than dust)