## G1 - require statement can be chained using && operator

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L65 and L66 can be chained in a single require() statement as such `require(msg.value >= minAmount && msg.value <= maxAmount, "invalid amount");` (error could be a custom error or even something other than `invalid amount`) so that when the line executes most of the times it saves gas and does not check second case.
The reason for adding minAmount first is because assuming the maxAmount is 200 ETH and there will be only rare cases that a person exceeds the maxAmount threashold but there is high possibility that many people will send less value.

Also, it is recommended to have the check and perform the interaction of this specific case of front end side in case the amount is entered by the user is less than minAmount then ask him to enter more and vice versa.

## G2 - for loop can be chained in one for loop

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L63
for loop can be chained in a single for loop saving loads of gas
the only things i changed is optimized variables which store default values again and merged both the for loop in one

Code :

```solidity
function stake() external payable {
  require(pauseStaking == false, "staking is paused");
  require(msg.value >= minAmount && msg.value <= maxAmount, "invalid amount"); // This is taken from G1 of my report

  uint256 underlyingValue;
  uint256 totalStakeValueEth; // total amount of derivatives worth of ETH in system

  // Getting underlying value in terms of ETH for each derivative
  for (uint i; i < derivativeCount; i++) {
    underlyingValue +=
      (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
        derivatives[i].balance()) /
      10 ** 18;

    uint256 weight = weights[i];
    IDerivative derivative = derivatives[i];
    if (weight == 0) continue;
    uint256 ethAmount = (msg.value * weight) / totalWeight;

    // This is slightly less than ethAmount because slippage
    uint256 depositAmount = derivative.deposit{ value: ethAmount }();
    uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
      depositAmount
    ) * depositAmount) / 10 ** 18;
    totalStakeValueEth += derivativeReceivedEthValue;
  }

  uint256 totalSupply = totalSupply();
  uint256 preDepositPrice; // Price of safETH in regards to ETH
  if (totalSupply == 0)
    preDepositPrice = 10 ** 18; // initializes with a price of 1
  else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

  // mintAmount represents a percentage of the total assets in the system
  uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;
  _mint(msg.sender, mintAmount);
  emit Staked(msg.sender, msg.value, mintAmount);
}

```

## G3 - Can return without storing to the variable

Here, we are storing the variable in memory and then returning that variable but we can save some gas simply by returning it.

case 1- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L79
`return wstEthBalancePost - wstEthBalancePre;`

case 2- https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L201
`return rethBalance2 - rethBalance1;`

the same can be done

## G4- Writing code for the functions which is already present

case 1 - rethAddress() is already defined https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L66. So we don't need to write it again and store it in a variable if it is used just once in a function we don't need to define it in line https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L121 instead we can directly use it in https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L129. Similar instances in deposit function

case 2 - balance()
When checking the balance in following lines we don't need to write the full functionality increasing the bytecode size and causing deploy size, we can already the function declared in L93 `balance()`
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L74 and in L74
replace `IWStETH(WST_ETH).balanceOf(address(this));` with balance()
