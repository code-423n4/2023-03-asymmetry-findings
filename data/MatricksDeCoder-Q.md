#### NC-1 Unlocked Pragma 

-Description => Contracts in scope Reth.sol SfrxEth.sol WstEth.sol SafeETH.sol make use of floating pragma  
-Impact => This can lead to problems with different versions being used for testing, production etc as a floating pragma implies any within range may be used could lead to outdated compiler, versions with different bugs etc
-POC => SWC103 https://swcregistry.io/docs/SWC-103 
-Tools Used => Manual 
-Recommendation => It is recommended to lock the pragma. Make use of pragma solidity 0.8.13 remove caret

#### NC-2 Solidity Version 

-Description => Contracts in scope Reth.sol SfrxEth.sol WstEth.sol SafeETH.sol make use of an older solidity version 
-Impact => Older versions may have bugs that are fixed in more recent versions which may impact compiler and or project 
-POC => https://swcregistry.io/docs/SWC-102 
-Tools Used => Manual 
-Recommendation => It is recommended to use latest versions of solidity e.g 0.8.17 0.8.18 etc

#### NC-3 Use Scientific Notation 

-Description => Contracts in scope Reth.sol SfrxEth.sol WstEth.sol SafeETH.sol make use of exponentiation based on multiplication for large numbers e.g maxSlippage = (1 * 10 ** 16); // 1% in lines of code below 
https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L44 

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L38

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L35

There are many other instances for division in contracts above where e.g 10**36, 10**18, etc are used


-Impact => This can be prone to errors e.g 1*10*18 when needed was 1*10**18. Arguably scientific notation is more readable compared to above 
-POC => https://docs.soliditylang.org/fr/latest/types.html 
-Tools Used => Manual 
-Recommendation => It is recommended to use scientific notation e.g 1*10**18 => 1e18 in all relevant instances 

#### NC-4 Natspec Code Commenting

-Description => Contracts in scope Reth.sol SfrxEth.sol WstEth.sol SafeETH.sol do not make use of consistent detailed Natspec commenting especially for functions with parameters e.g 

(a) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L81
Does not detail @return amountOut the amount that etc etc ...

(b) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L106
Does not detail @param _amount

(c) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L119
Does not detail @return a Boolean detailing if success of etc etc etc...

(d) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L155
Does not detail @return e.g An amount that is etc etc etc 

(e) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L210
Does not detail @return e.g Pool price etc etc 

(f) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L50 
Does not detail @param _slippage 

(g) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L93
Does not detail @return e.g A uint value representing the amount etc etc 

(h) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L56
Does not detail @param _amount e.g The amount that etc 

There are many other instances but those are used to be representative of cases where complete NatSpec is ideal to improve code quality, readability etc. 

-Impact => This can reduce the ability or other devs, users, auditors to understand the functions 
-POC => https://docs.soliditylang.org/en/v0.8.17/natspec-format.html 
-Tools Used => Manual 
-Recommendation => It is recommended to fix all relevant function instances to make use of rich Natspec code commenting which allows for solid documentation of functions, how they work, its paramaters and return values to help with code readability and maintainability 

#### NC-5 Function Parameters Use of UnderScore 

-Description => Contracts in scope Reth.sol SfrxEth.sol WstEth.sol SafeETH.sol make use _ underscore in front of function parameters as best practise. However certain instances as below are not consistent with other code 

(a) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L107
Use of amount instead of _amount 

-Impact => This can lead to inconsistencies in code 
-POC => Inconsistency  
-Tools Used => Manual 
-Recommendation => It is recommended to make use of underscore _ before all function parameters 

#### NC-6 Implicit use of uint

-Description => There are instances where uint is used without specifying which type e.g uint256 uint8 in the contracts e.g 
(a) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L26
Followed by lines 27,28, 
(b) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71
Followed by line 84,140,147 in similar in for loop
(c) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L92
(d) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L203
Followed by line 204
(e) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L171
(f) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L241

-Impact => This can lead to readability, maintenance or even make code prone to error if incorrect casting of values occurs 
-POC => Recommended Best Practice 
-Tools Used => Manual 
-Recommendation => It is recommended to always be specific with the uint types e.g uint256. Fix all instances mentioned and other similar hay may not have been mentioned

#### NC-7 Comparison Boolean Constant 

-Description => There are instances as in example below where Boolean comparisons are made 
(a) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L109
require(pauseUnstaking == false, "unstaking is paused");

(b) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L64
require(pauseStaking == false, "staking is paused");

-Impact => This is waste of gas, and it may even introduce errors if compare incorrect value when wanted to compare with true but compares with false 
-POC => Best Practise
-Tools Used => Manual 
-Recommendation => It is recommended to not compare with other boolean for value already boolean e.g 
require(pauseUnstaking == false,...) => require(!pauseUnstaking,....)

#### Low-1 Missing Events Critical Parameter Changes  

-Description => There are critical functions, parameter updates performed by privileged roles that do not emit events
(a) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L58
Slippage is changed but no event is emitted 
(b) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51
(c) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48


-Impact => This can lead to lack of transparency, lack of monitoring, insufficient information o off chain tooling that may depend on emission of these critical events. 
-POC => Best Practise
-Tools Used => Manual 
-Recommendation => It is recommended to emit events for critical parameter updates like e.g slippage. Fix for above instances or any other relevant that are useful to report changes to contract functionality 

#### Low 2 Function parameters lacking sanity checks 

-Description => Function parameter values like _slippage are lacking sanity checks to bound or control values 
(a) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L58
Slippage is changed but no event is emitted 
(b) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L51
(c) https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L48
-Impact => This can lead to  bugs if values too large or too small or with incorrect exponent or incorrect scale relative to where used to calculations can lead to errors 
-POC => Best Practise
-Tools Used => Manual 
-Recommendation => It is recommended to validate that all parameters are within safe bounds 




