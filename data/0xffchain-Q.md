####  [1] Validation for sufficient funds 

The `unstake()` function in `SafEth.sol` does not check that the user balance is sufficient to deduct amount  `_safEthAmount`  supplied as function input, 
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L108

        function unstake(uint256 _safEthAmount) external {
        require(pauseUnstaking == false, "unstaking is paused");
        uint256 safEthTotalSupply = totalSupply();
        uint256 ethAmountBefore = address(this).balance;

        for (uint256 i = 0; i < derivativeCount; i++) {
            // withdraw a percentage of each asset based on the amount of safETH
            uint256 derivativeAmount = (derivatives[i].balance() *
                _safEthAmount) / safEthTotalSupply;
            if (derivativeAmount == 0) continue; // if derivative empty ignore
            derivatives[i].withdraw(derivativeAmount);
        }
        _burn(msg.sender, _safEthAmount);
        uint256 ethAmountAfter = address(this).balance;
        uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;
        // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
            ""
        );
        require(sent, "Failed to send Ether");
        emit Unstaked(msg.sender, ethAmountToWithdraw, _safEthAmount);
       }


until the  `_burn(address, uint)`  function, which by then sufficient gas been wasted.  there should be a require function at the start of the function which checks like so: 

    
     function unstake(uint256 _safEthAmount) external {
      require(pauseUnstaking == false, "unstaking is paused");
      require(balanceOf(msg.sender)>= _safEthAmount, 'message here') 
     ..............
     }


#### [2] Validation for TotalSupply 

In the rebalance function in SafEth.sol, 

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L138

    function rebalanceToWeights() external onlyOwner {
        uint256 ethAmountBefore = address(this).balance;
        for (uint i = 0; i < derivativeCount; i++) {
            if (derivatives[i].balance() > 0)
                derivatives[i].withdraw(derivatives[i].balance());
        }
        uint256 ethAmountAfter = address(this).balance;
        uint256 ethAmountToRebalance = ethAmountAfter - ethAmountBefore;

        for (uint i = 0; i < derivativeCount; i++) {
            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
                totalWeight;
            // Price will change due to slippage
            derivatives[i].deposit{value: ethAmount}();
        }
        emit Rebalanced();
    }

there should an assert that checks if there is value staked in the system before continuing the process, because the rebalancing mechanism only comes into play when there is asset staked in the protocol. If there is no asset then this should fail as there is nothing to rebalance. 


    function rebalanceToWeights() external onlyOwner {
        require(totalSupply() > 0, 'message');
        uint256 ethAmountBefore = address(this).balance;
        .............
    }

An attempt is made on this at L#148 
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L148

     (weights[i] == 0 || ethAmountToRebalance == 0) continue
But this can be done earlier 


#### [3]  Validation for contract Address

In the `addDerivative()` function , the first parameter of `_contractAddress` is not validated when passed in, for example the allowed user might pass a zero address and successfully and they would be no way to delete this, except to disable the weight on finding out.
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L182

a validation like so should be included: 

    function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
	require(_contractAddress != address(0), 'message');
	.........................
	}


#### [4]  Unothorized access to stock eth. 

The withdraw function on Reth.sol allows anyone the ability to remove eth stock in the contract, 
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L107

       function withdraw(uint256 amount) external onlyOwner {
		   RocketTokenRETHInterface(rethAddress()).burn(amount);
		   // solhint-disable-next-line
		   (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
		   ""
		   );
		   require(sent, "Failed to send Ether");
       }
	


Steps to replicate: 
- User A sends 5eth directly to Reth.sol, through the recieve function. The balance of the Reth.sol contract is then 5eth. 
- User B has 2000 SafEth token and wants to unstake 
- User B calls unstake on SafEth.sol, 
- It attempts to withdraws user B's percentage of Eth locked in all deriavatives.When it calls withdraw on Reth.sol, it withdraws the User B's Eth locked on Reth protocol, which sends let us assume 20Eth to the Reth.sol contract. The current balance of the Reth.sol contract now is 25eth, the User B is then sent the entire balance of the Reth.sol which is now 25Eth, as opposed to 20Eth.  

Mitigation: 
```java
 function withdraw(uint256 amount) external onlyOwner {
	   	uint EthAmountBefore = address(this).balance;
		RocketTokenRETHInterface(rethAddress()).burn(amount);
		uint EthAmountAfter = address(this).balance;
		   
		   uintEthAmountToSend = EthAmountAfter - EthAmountBefore;
		   // solhint-disable-next-line
		   (bool sent, ) = address(msg.sender).call{value: EthAmountToSend }(
		   ""
		   );
		   
		   require(sent, "Failed to send Ether");
       }
```

#### [5] Inconsistent naming 

In temporary storing balances before and after making external calls, there is an inconsistent use of nomenclature in performing the same operation on multiple contracts. 

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L94


    uint256 sfrxBalancePre = IERC20(SFRX_ETH_ADDRESS).balanceOf(address(this));
 	frxETHMinterContract.submitAndDeposit{value: msg.value}(address(this));
 	uint256 sfrxBalancePost = IERC20(SFRX_ETH_ADDRESS).balanceOf(address(this));

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L197

	uint256 rethBalance1 = rocketTokenRETH.balanceOf(address(this));
	rocketDepositPool.deposit{value: msg.value}();
	uint256 rethBalance2 = rocketTokenRETH.balanceOf(address(this));
	
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L121

	uint256 ethAmountBefore = address(this).balance;
	..........................
	uint256 ethAmountAfter = address(this).balance;
	uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;

A more consistent naming nomenclature should be used for these operations. For example either going for the before and after suffixes or the pre and post suffixes and then using these all around the codebase as may warrant. 


#### [6] Inconsistent return values. 

The return value of function `deposit` on sfrxEth.sol returns the direct aritmetic balance of the post and pre balance like so: 

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L105


 	function deposit() external payable onlyOwner returns (uint256) {
 	....................
	return sfrxBalancePost - sfrxBalancePre;
	}

 but in similar operations on both Reth.sol and WstEth.sol it returns a variable which stores the result of the aritmetic operation.
 
 https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L202
 
	function deposit() external payable onlyOwner returns (uint256) {
	....................
	uint256 rethMinted = rethBalance2 - rethBalance1;
	return (rethMinted);
	}

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L80

	function deposit() external payable onlyOwner returns (uint256) {
	....................
		uint256 wstEthAmount = wstEthBalancePost - wstEthBalancePre;
		return (wstEthAmount);
	}

A more consistent design style should be used in all scenarios this is needed in the codebase. 


#### [7] Inconsistent naming 

The variable containing the result of the pre and post balance operation in both Wst.Eth and Reth.sol should both maintain same name for code consistency as they perform same action, the current names make it seem that one performs a different operation from the other. 

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L79

	function deposit() external payable onlyOwner returns (uint256) {
		....................
		uint256 wstEthAmount = wstEthBalancePost - wstEthBalancePre;
		...................
	}
	
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L201

	function deposit() external payable onlyOwner returns (uint256) {
	...................
		uint256 rethMinted = rethBalance2 - rethBalance1;
		...................
	}


#### [8] Inconsistent error reporting 

There is no error here that returns on failure to deposit derivative into the SfrxEth.sol contract. 

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L94

unlike is available in the other derivatives 
WstEth.sol https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L77

and 

Reth.sol.https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L200

Although since this is a contract to contract call, and would revert on failure, there should be a message from this deriavative about the failure and the reason it failed, just like in the other two derivative contracts. 

#### [9] Inconsistent error report messages 

The same operation in both WstEth.sol and Reth.sol, returns two different error messages. 

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L77

    function deposit() external payable onlyOwner returns (uint256) {
	....................
        require(sent, "Failed to send Ether");
	....................
    }

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L200

	function deposit() external payable onlyOwner returns (uint256) {
	..................
		require(rethBalance2 > rethBalance1, "No rETH was minted");
	.................
	}
the former returns "Failed to send Ether" while the later returns "no rEth was minted".  There should be consistency in the error messages, so as not to confuse the user and for better UX. Consider using the same error messages for these operations. 


#### [10] Validation of input parameter 

The adjustWeight function in SafEth.sol does not validate the _derivativeIndex  parameter that is passed to it, 

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165

    function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
        weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];
        totalWeight = localTotalWeight;
        emit WeightChange(_derivativeIndex, _weight);
    }

there should be a check like so 

    function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
		require (_derivativeIndex <  derivativeCount,  "message");
       .......................
    }
	

this way a wrong index that exceeds the already supplied indexes is not included, for example if a wrong index of 7 is supplied when there is only 3 derivatives in this contract, it will mean that the that new derivative would never be reached when looping  for actions that take weights into consideration like so 

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L84

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171