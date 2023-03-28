https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol and others 

10 ** 18 is too big percentage precision.

I suggest to replace

line35:       maxSlippage = (1 * 10 ** 16); // 1%
line60:       uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;

by something like  

line35:       maxSlippage = (1 * 10 ** 2); // 1%
line60:       uint256 minOut = (stEthBal * (10 ** 4 - maxSlippage)) / 10 ** 4


which reduces computational costs.