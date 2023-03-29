
# Low Risk and Non-Critical Issues

Some of the opportunities identified for improving low severity issues throughout the codebase of Asymmetry protocol are categorised into 03 main areas; with further multiple instances in each of the category.


# [L-01] Minting tokens to the zero address should be avoided (01 Instance)

Address(0) check is missing in function, consider applying check to ensure tokens aren’t minted to the zero address.

Link to the code:
1.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L99


# [L-02] Add Timelock to Critical Parameter Change (08 Instances)

It is better to give users time to react and adjust to critical changes with a mandatory time window between them.
Timelock also prevent front running to make malicious changes just ahead of any major event or incident that can take place due to unfortunate timing of changes.

Link to the code:
1.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L48
2.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48
3.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L36
4.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51
5.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165
6.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L214
7.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L223
8.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L42


# [N-01] Floating of Pragma (04 Instances)
Locking pragma version ensures contracts are not being deployed on an outdated compiler version.

Link to the code:
1.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L2
2.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L2
3.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L2
4.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L2


# [N-02] Empty blocks should be removed (04 Instances)

Avoid using code blocks or use them for emitting events.

Link to the code:
1.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L97
2.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L126
3.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L246
4.	https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L244