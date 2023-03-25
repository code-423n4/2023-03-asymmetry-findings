# asymmetry

### [L1] one step owner transferRole transfer actions done in a single-step manner are dangerous

## Impact
Inheriting from OpenZeppelin's Ownable contract means you are using a single-step ownership transfer pattern. If an admin provides an incorrect address for the new owner this will result in none of the onlyOwner marked methods being callable again. The better way to do this is to use a two-step ownership transfer approach, where the new owner should first claim its new rights before they are transferred.

## Proof of Concept
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L5

`import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";`

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L6
`import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";`

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L9

```
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L13
```
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
```

## Tools Used
Manual Review
## Recommended Mitigation Steps
Use OpenZeppelin's Ownable2Step instead of Ownable

### [L2] Missing admin input sanitization admin

## Impact
Missing maximum value check that is set by the contract owner. This issue can allow the contract owner to set big sllippage or big weight even  over 100. It will cause unexpected operactions in protocol.

## Proof of Concept
1.https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165-L174

For example, total weight should be 100 = 50/25/25 (3 derivatives). But there is no maximum threshold validation.
```
    function adjustWeight(
        uint256 _derivativeIndex,
        uint256 _weight
    ) external onlyOwner {
        weights[_derivativeIndex] = _weight;
        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];  //@audit no maximum threshold validation.
        totalWeight = localTotalWeight;
        emit WeightChange(_derivativeIndex, _weight);
    }
```

2.https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L192

For example, total weight should be 100 = 50/25/25 (3 derivatives). But there is no maximum threshold validation.
```
    function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress);
        weights[derivativeCount] = _weight;
        derivativeCount++;

        uint256 localTotalWeight = 0;
        for (uint256 i = 0; i < derivativeCount; i++)
            localTotalWeight += weights[i];   //@audit no maximum threshold validation.
        totalWeight = localTotalWeight;
        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
    }
 
```
3. function setMaxSlippage() - No maximum threshold validation.
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202
```
    function setMaxSlippage(
        uint _derivativeIndex,
        uint _slippage
    ) external onlyOwner {
        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
        emit SetMaxSlippage(_derivativeIndex, _slippage);
    }
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51
```
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48
```
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58
```
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```
## Tools Used
Manual Review
## Recommended Mitigation Steps
Add maximum threshold validation.
 
### [L3] Missing events for critical parameter changing operations by onlyOwner

## Impact
onlyOwner only functions that change critical parameters should emit events. There are also no any events log deposit, withdraw, etc. in SfrxEth.sol, Reth.sol and WstEth.sol.

## Proof of Concept
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51
```
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48
```
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58
```
    function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
```


## Tools Used
Manual Review
## Recommended Mitigation Steps
Consider emitting events when these addresses/values are updated by onlyOwner.
 
### [L4] sqrtPriceLimitX96 is set to 0 to disable slippage protection

## Impact
sqrtPriceLimitX96 is set to 0 to disable slippage protection in the Reth.sol contract. 

## Proof of Concept
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L99
```
    function swapExactInputSingleHop(
        address _tokenIn,
        address _tokenOut,
        uint24 _poolFee,
        uint256 _amountIn,
        uint256 _minOut
    ) private returns (uint256 amountOut) {
        IERC20(_tokenIn).approve(UNISWAP_ROUTER, _amountIn);
        ISwapRouter.ExactInputSingleParams memory params = ISwapRouter
            .ExactInputSingleParams({
                tokenIn: _tokenIn,
                tokenOut: _tokenOut,
                fee: _poolFee,
                recipient: address(this),
                amountIn: _amountIn,
                amountOutMinimum: _minOut,
                sqrtPriceLimitX96: 0    //@audit disable slippage protection
            });
        amountOut = ISwapRouter(UNISWAP_ROUTER).exactInputSingle(params);
    }
```

## Tools Used
Manual Review
## Recommended Mitigation Steps
Consider to set sqrtPriceLimitX96.

### [L5]  Without contract integrity check

## Impact
It is recommended to add a "checkContract" function to verify the integrity of the contract code. 

## Proof of Concept
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L186
```
    function addDerivative(
        address _contractAddress,
        uint256 _weight
    ) external onlyOwner {
        derivatives[derivativeCount] = IDerivative(_contractAddress //@audit no contract check
        weights[derivativeCount] = _weight;
```


## Tools Used
Manual Review
## Recommended Mitigation Steps
It is recommended to add a "checkContract" function.
Example:

```
    function checkContract(address _account) internal view {
        require(_account != address(0), "Account cannot be zero address");

        uint256 size;
        // solhint-disable-next-line no-inline-assembly
        assembly { size := extcodesize(_account) }
        require(size > 0, "Account code size cannot be zero");
    }

```