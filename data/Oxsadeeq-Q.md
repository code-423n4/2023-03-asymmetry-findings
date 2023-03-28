                No checks for amount being a zero value
Vulnerability: There is no check for if ethAmountToWithdraw is a zero value, if it is a zero value,0 Ether is been sent to the address which cost unecessary gas.
P.O.C:for (uint256 i = 0; i < derivativeCount; i++) {
            // withdraw a percentage of each asset based on the amount of safETH
            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
            if (derivativeAmount == 0) continue; // if derivative empty ignore
            derivatives[i].withdraw(derivativeAmount);
        }
        _burn(msg.sender, _safEthAmount);//my questions unanswered is when a safeEth eth is burnt does it eth to the contract?
        uint256 ethAmountAfter = address(this).balance;
        uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;
In the contract if derivativeAmount is 0, ether is not sent to the account which means ethAmountAfter==ethAmountBefore which when subtracted is zero and then 0 ether is sent to the msg.sender.
Mitigation:Add require statement to check if ethAmountBefore is zero
//require(ethAmountBefore>0,"Insufficient ether");
Function Affected:https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol
Unstake().

 