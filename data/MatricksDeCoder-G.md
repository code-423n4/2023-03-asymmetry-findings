#### Gas-1 Use Assembly for Simple Getter and Setter functions 

It may be possible to save on gas costs by using Yul assembly. Although generally this can be applied to all functions. This is not ideal as it can hinder readability, auditability of code, introduce errors or harm code as Yul Assembly will lack in-build safety checks. 

However simple getter and setter functions normally have 1 line code, touch storage with not much critical error raising issues and can be rewritten in yul without impacting readability. e.g 
(a) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L44
(b) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L59
(c) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L38
(d) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L52
(e) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L49
Above used to show examples where state variable is set using 
maxSlippage = (1 * 10 ** 16); which can be rewritten 
assembly {
    sstore(maxSlippage.slot,1e16);
}
```
function setMaxSlippage(uint256 _slippage) external onlyOwner {
        maxSlippage = _slippage;
    }
Can be rewritten 
function setMaxSlippage(uint256 _slippage) external onlyOwner {
        assembly {
           sstore(maxSlippage.slot,_slippage);
        }
}
```
Above don't impact readability or prone to assembly errors 

#### Gas-2 Save hashed values as constants  

Hashing is very expensive in Solidity keccak256 e.g 30 gas plus additional cost per each word of input data. There are many instances where values are hashed that are non changing strings, sometimes 
(a) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L69
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L191                       
```
keccak256(abi.encodePacked("contract.address", "rocketTokenRETH"))
Can be saved as e.g ->
bytes32 private constant ROCKET_TOKEN_ETH = 0xe3744443225bff7cc22028be036b80de58057d65a3fdca0a3df329f525e31ccc;
So that part were used is
RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(ROCKET_TOKEN_ETH)
```

(b) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L124
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L161
```
keccak256(abi.encodePacked("contract.address", "rocketDepositPool"))
Can be saved as e.g ->
bytes32 private constant ROCKET_DEPOSIT_POOL = 0x65dd923ddfc8d8ae6088f80077201d2403cbd565f0ba25e09841e2799ec90bb2;
So that part were used is
RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(ROCKET_DEPOSIT_POOL)
```

(c) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L135
```
keccak256(abi.encodePacked("contract.address", "rocketDAOProtocolSettingsDeposit"))
Can be saved as e.g ->
bytes32 private constant ROCKET_DAO_SETTING_POOL = 0x876d8a498b5341a6b5897a8239bfb0347e2b8d81fe9401d355c1e4b0ecebedaa;
So that part were used is
RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(ROCKET_DAO_SETTING_POOL)
```
Or whatever correct values of the keccak hashing can be stored as constants


#### Gas-3 Cache address(this) value 

There are several instances in a function where address(this) is evaluated more than once or over and over again. This incurs gas costs fo dynamic cost e.g 100 [or whatever value not sure] gas over and over again repeatedly. It is recommended to evaluate value once and cache in a memory variable e.g 
(a) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L197
Line 197 and again line 199 

(b) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L63
From line 63, 64, 67, 84 in same function withdraw()

(c) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L94
From line 99,101,103,used many times in same function deposit() 

(d) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L58
Line 58, 63 in function withdraw() 

(e) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L74
Line 74,78 in same function deposit() 

(f) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L111
Line 111,121 in function unstake() 

(g) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L139
Line 139,144 in same function rebalanceToWeights() 

Gas savings for the above mentioned or any other similar relevant cases can be saved by caching the value as 
```
address memory _this = addres(this);
Then reuse _this where appropriate

``


