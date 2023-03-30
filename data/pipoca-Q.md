## 1) SafEth.sol - Validation Missing in adjustWeight()

The adjustWeight() function accepts a _derivativeIndex parameter, which represents the index of the derivative for which you want to update the weight. However, there are no checks to ensure that the provided index is within the valid range of the derivatives array. This lack of input validation could lead to unexpected behavior, such as accessing or modifying an incorrect derivative or even causing out-of-bounds access.

To address this issue, you should add checks within the adjustWeight() function to validate that the provided _derivativeIndex is within the valid range of the derivatives array. One way to do this is by using a require statement to check if the provided index is less than the derivativeCount variable

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165

## 2) SafEth.sol - Error-prone manual array handling

The issue with error-prone manual array handling in the functions addDerivative() and adjustWeight() is that they manage indices and weights in the arrays through direct manipulation, which can lead to mistakes and errors. 

In the addDerivative() function, the new derivative is added to the derivatives array at the position indicated by derivativeCount. Similarly, the function also adds a new weight to the weights array. This manual handling of indices and array growth can be prone to errors, such as incorrect index updates or inconsistencies between the two arrays.

In the adjustWeight() function, you directly update the weight at the specified index in the weights array. This approach can be error-prone, as it does not validate the index and can lead to out-of-bounds access, as mentioned earlier.

To mitigate these issues, you can use data structures like mappings to store the derivatives and their respective weights. By using mappings, you can simplify the management of derivatives and weights, reducing the risk of errors. Here's an example of how you can refactor the contract to use mappings:

`// Replace the arrays with mappings
mapping(uint256 => IDerivative) public derivatives;
mapping(uint256 => uint256) public weights;`

`// Update the addDerivative() function
function addDerivative(address _contractAddress, uint256 _weight) external onlyOwner {
    derivatives[derivativeCount] = IDerivative(_contractAddress);
    weights[derivativeCount] = _weight;
    derivativeCount++;
    // ... rest of the function
}`

`// Update the adjustWeight() function
function adjustWeight(uint256 _derivativeIndex, uint256 _weight) external onlyOwner {
    require(_derivativeIndex < derivativeCount, "Invalid index provided");
    weights[_derivativeIndex] = _weight;
    // ... rest of the function
}`

By using mappings instead of arrays, you can improve the contract's maintainability and reduce the potential for errors related to manual array handling.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L165
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182

## 3) SafEth.sol – No checks for same derivative

Although only the owner has the permission to add derivatives using the addDerivative() function, it's still essential to ensure that the same derivative is not added multiple times, as it could lead to unexpected behavior and complicate the management of derivatives and weights.

To address this issue, introduce a check in the addDerivative() function to ensure that the provided _contractAddress has not been added previously as a derivative. One way to implement this is by maintaining a mapping that keeps track of all the added derivative addresses.


`// Add a mapping to store added derivatives
mapping(address => bool) private addedDerivatives;`

`// Update the addDerivative() function
function addDerivative(address _contractAddress, uint256 _weight) external onlyOwner {
    require(!addedDerivatives[_contractAddress], "Derivative already added");

    addedDerivatives[_contractAddress] = true;
    derivatives[derivativeCount] = IDerivative(_contractAddress);
    weights[derivativeCount] = _weight;
    derivativeCount++;
    // ... rest of the function
}`

With this implementation, when the addDerivative() function is called, it checks if the provided _contractAddress already exists in the addedDerivatives mapping. If it does, the function will revert with the error message "Derivative already added." This prevents the same derivative from being added multiple times and ensures that each entry in the derivatives mapping is unique.

Adding this check helps to maintain the integrity of the contract and reduces the risk of duplicated entries or unexpected results.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182

## 4) SafEth.sol - No check for valid derivative address

In the addDerivative function, the provided _contractAddress is added as a new derivative without any validation. This means that any address can be added, even if it's not a valid contract address or if the contract at the address does not implement the expected IDerivative interface. The lack of validation could lead to unintended consequences in the following ways:

If the added address is not a contract address, subsequent calls to the functions of the IDerivative interface will fail, potentially causing the stake and unstake functions to revert or behave unexpectedly.

If the added address is a contract but does not implement the IDerivative interface, the contract's functions might have different behavior than expected. This could cause unpredictable results when interacting with the derivative through the SafEth contract.

If the added address is a malicious contract, it could exploit the lack of validation to manipulate the SafEth contract's behavior, potentially causing loss of funds or other undesirable consequences.

To mitigate these risks, you can add validation checks in the addDerivative function to ensure that the provided address is a valid contract and that it implements the expected IDerivative interface. You can do this by:

Checking if the address is a contract: You can use the Address library from OpenZeppelin to check if the provided address is a contract. The Address.isContract function returns true if the address is a contract.

`require(Address.isContract(_contractAddress), "Address must be a contract");`

Checking if the contract implements the IDerivative interface: You can use the ERC165 standard for interface detection. The IDerivative interface should inherit from IERC165, and each contract implementing the IDerivative interface should register the interface ID using _registerInterface. Then, in the addDerivative function, you can use the supportsInterface function to check if the contract supports the IDerivative interface.

`require(
    IDerivative(_contractAddress).supportsInterface(type(IDerivative).interfaceId),
    "Contract must support IDerivative interface"
);`

By adding these validation checks, you can help ensure that only valid and expected derivative contracts are added to the index fund, reducing the risk of unintended consequences.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L182

## 5) SafEth.sol – Insufficient Information in the Rebalanced Event Emission

To enhance the contract's transparency and provide better insights to off-chain systems or applications monitoring the contract's events, it is recommended to include relevant data in the Rebalanced event.

Update the Rebalanced event definition to include the address of the entity initiating the rebalance, the total amount of ETH rebalanced, and the new balances of each derivative:


`event Rebalanced(
    address indexed by,
    uint256 totalRebalancedEth,
    uint256[] newDerivativeBalances
);`

In the rebalanceToWeights() function, create an array to store the new balances of each derivative after rebalancing:

`uint256[] memory newDerivativeBalances = new uint256[](derivativeCount);`

Update the rebalanceToWeights() function's loop to store the new balances of each derivative:

`for (uint i = 0; i < derivativeCount; i++) {
    // ...
    derivatives[i].deposit{value: ethAmount}();
    newDerivativeBalances[i] = derivatives[i].balance();
}`

Update the event emission in the rebalanceToWeights() function to include the relevant data:

`emit Rebalanced(msg.sender, ethAmountToRebalance, newDerivativeBalances);`

By including the relevant data in the Rebalanced event, you can provide valuable information to off-chain systems or applications that monitor the contract's events, improving overall transparency and visibility into the contract's operation.

The rebalanceToWeights() function is designed to rebalance the derivatives in the index fund by withdrawing and redepositing the assets according to their set weights. The function emits a Rebalanced() event, which lacks relevant information about the rebalancing process.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L138

## 6) SafEth.sol – technically incorrect comment 

Technically incorrect comment on line 80. Should say 1 eth, not 1 without label.

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L80

## 7) WstEth.sol - Unused _amount in ethPerDerivative function can be removed

   function ethPerDerivative(uint256 _amount) public view returns (uint256) {
        return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);
    }


https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#LL86


## 8) No maximum limit set for maximum slippage

The SafEth , WstEth, SfrxEth, and Reth contracts have a maxSlippage variable. However, no maximum limit is enforced for the maxSlippage variable when it is updated using the setMaxSlippage function in these contracts.

To mitigate potential issues caused by an extremely high slippage value, it is recommended to enforce a maximum limit for the maxSlippage variable when updating it in the setMaxSlippage function for all relevant contracts. 
This can be accomplished by adding a require statement to verify that the new slippage value is within an acceptable range. For example:

`require(_slippage <= MAX_SLIPPAGE_LIMIT, "Slippage value exceeds maximum limit");`

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L202
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L48
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L51
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L58




