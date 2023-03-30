[M-16] Use of Solidity version 0.8.13 which has two known issues applicable to PuttyV2
intializable inherites ----------> AddressUpgradeable

# Findings Summary

| ID     | Title                                                            | Severity |
| ------ | ---------------------------------------------------------------- | -------- |
| [L-01] | Gas griefing/theft is possible on unsafe external call           | low      |
| [L-02] | event missed in `setMaxSlippage()` function may cause fund loose | low      |

# [L-01] contracts should inherit their interfacese

## Description

return data (bool success,) has to be stored due to EVM architecture, if in a usage like below, ‘out’ and ‘outsize’ values are given (0,0) . Thus, this storage disappears and may come from external contracts a possible Gas griefing/theft problem is avoided

## context

There are 5 instances of the topic.

```solidity

file: contracts/SafEth/SafEth.sol

 // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw}(
            ""
        );
        require(sent, "Failed to send Ether");
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L123-L127

```solidity
file: contracts/SafEth/derivatives/Reth.sol

 (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
    require(sent, "Failed to send Ether");
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L110-L112

```solidity
file: contracts/SafEth/derivatives/SfrxEth.sol

 // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
        require(sent, "Failed to send Ether");
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L84-L87

```solidity
file: contracts/SafEth/derivatives/WstEth.sol

 (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
            ""
        );
        require(sent, "Failed to send Ether");
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L63-L66

```solidity
file: contracts/SafEth/derivatives/WstEth.sol

 (bool sent, ) = WST_ETH.call{value: msg.value}("");
        require(sent, "Failed to send Ether");
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L76-L77

## Recommendations

change the calls method above to something like this

```solidity
            assembly {
                success := call(gas(), dest, amount, 0, 0)
            }

require(success,"transfer failed");
}
```

# [L-02] event missed in `setMaxSlippage()` function may cause fund loose

## Description

there is event missed in function `setMaxSlippage()`, the owner can set maxSlippage to more than 1% and when this happen users don't know about the new slippage that set by the user and may cause fund loose during stake or any money trading on the protocol(like swapping) because of lack of event in this function. i set this as low risk because i don't see this will cause the protocol loose fund directly and let the judge decide about the severity.

## context

There are 5 instances of the topic.

```solidity

file: contracts/SafEth/SafEth.sol


    /**
        @notice - Sets the max slippage for a certain derivative index
        @param _derivativeIndex - index of the derivative you want to update the slippage
        @param _slippage - new slippage amount in wei
    */
    function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
    ) external onlyOwner {
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
    }
```

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L202-L208

## Recommendations

I reccomend adding event to `setMaxSlippage()` function so the user can be aware of the updated slippage updated/set by the owner:

`event UpdateSlippage()`
