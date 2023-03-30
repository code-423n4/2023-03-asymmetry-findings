

| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| O | Ordinary | Often found issues |

| Total Found Issues | 10 |
|:--:|:--:|

### Non-Critical 

| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | Floating Pragma | 5 |
| [N-02] | Same user can stake more than 200 ETH  | 1 |
| [N-03] |    derivatives[i].withdraw(derivativeAmount);   sending Ether to safETH contract  if  anyone call the unstake function  then it might burn the tokens in  derivative contracts | 1 |


### Refactor Issues 

| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [R-01] | Use uint instead of boolean in pauseUnstaking  and pauseStaking | 1 |
| [R-02] | If user doen't have sufficient safETH revert if  unstake() function is called  | 1 |


### Low Risk 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-01] | No nonreentrant modifier used in the  functions stake and unstake  | 1 |

### [R-0]  Use uint instead of boolean in pauseUnstaking  and pauseStaking;

It's better to use uint  instead of bool  ,  a really good example is openzeppelin re-entrancy guard , Also
boolean is way more expensive than  using uint ;
```js
   -   bool public pauseStaking; // true if staking is paused
   -   bool public pauseUnstaking; // true if unstaking is pause
```

eg: 
```js
 uint256 private constant notpaused = 1 ;
    uint256 private constant paused = 2;
    uint256 private stakelock ;
    uint256 private unstakelock ;
```

```js
// inside inizialer we innitialize it to paused 
stakelock = notpaused ;
 unstakelock = notpaused;
```

eg:
```js
 function setPauseUnstaking() external onlyOwner {
          if (unstakelock == paused){
            unstakelock = notpaused;
        }
        else {
             unstakelock = paused;
        }

```


### [R-1]  If user doen't have sufficient safETH revert if  unstake() function is called 

unstake funtion doesn't have any checks any user  can call unstake function not only  users who stake 
safETH.
    mapping (address => uint256) SafEthHoldersBalances;
    
   we can create a mapping to check whether user have sufficient balance to  unstake()
```js
    if(SafEthHoldersBalances[msg.sender]  < _safEthAmount ){

                        revert ("User isn't  a safETH hodler");

                       }
```

```js
  REVERT Error(reason: "User isn't  a safETH hodler")
   REVERT Error(reason: "User isn't  a safETH hodler")
```
 (In protocol It hasn't mentioned that  users  not allowed to transfer their safETH Tokens  so a user who already staked can transfer his safETH tokens and reciever can call unstake without staking )    

### [L-1] No nonreentrant modifier used in the  functions stake and unstake 

### Recommendations

use openzeppelin re-entrancy Guard and nonreentrant modifier 


## [N-0] Floating Pragma

**Description**: 
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

**Recommendation**
Consider locking the pragma version.

### BUGS IN SOLIDITY 0.8.13
Please consider using at least Solidity 0.8.15 instead of 0.8.13 
-  https://blog.soliditylang.org/2022/05/17/calldata-reencode-size-check-bug/
- https://blog.soliditylang.org/2022/06/15/inline-assembly-memory-side-effects-bug/

### [NC-01] Same user can stake more than 200 ETH 
 Same user can  stake more that 200 ETH 

```js
 it("USER1 STAKE 200 ether " , async function () {
      console.log("BALANCE BEFORE : %s" , await safEthProxy.balanceETHERS() )
           const a = ethers.utils.parseEther("200")
        const  s = await safEthProxy.connect(user).stake( {value: a }   )
        await s.wait();
    }
    )
it("USER1  again STAKE 200 ether " , async function () {
  console.log("BALANCE BEFORE : %s" , await safEthProxy.balanceETHERS() )
        const a = ethers.utils.parseEther("200")
     const  s = await safEthProxy.connect(user).stake( {value: a }   )
    await s.wait();
}
    )
```


```js

MINT : 200032451100303076621
      ✓ USER1 STAKE 200 ether 
MINT : 200032681248109301110
      ✓ USER1  again STAKE 200 ether 
```




### [N-3]  Add a  time limit  so that  users who stake  can call unstake after that   time limit 

***suggestion****

### [O-1] Use  ++i instead of i++
```js
   for (uint i = 0; i < derivativeCount; i++) 
```

### [O-2]  instead of x += y  use  x = x + y for state variables



### [N-2 ]   derivatives[i].withdraw(derivativeAmount);   sending Ether to safETH contract  if  anyone call the unstake function  then it might burn the tokens in  derivative contracts
So when we call stake function  , it will call deposit functions in each derivatives contracts , each derivatives contract have  an `onlyOwner`  which is the  `safETH` contract address , so when calll stake it will deposit  rETH    to RocktPool ... then I think it calls uniswap router and exhange ETH to WETH to  rETH msg.sender is Reth pool Token contract .....  similar way to other tokens 
```
rETH(0xae78736cd615f374d3085123a210448e74fc6393).transfer(to: [RocketPool], amount: 0.311470004114065856)
```
 so I tried calling the unstake() function  with  passing 1 ether as param  without staking any token ( 0 ) because  it has  no checks , so any one can call unstake with any amount   , 
function executes till 
 `` derivatives[i].withdraw(derivativeAmount); ``

```js
function withdraw(uint256 amount) external onlyOwner {
        RocketTokenRETHInterface(rethAddress()).burn(amount);
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}("");
        require(sent, "Failed to send Ether");
```

which call  withdraw in respective derivative contracts  
function unstake execute until  `_burn(msg.sender, _safEthAmount); (here msg.sender is Reth contract)
as there's no amount it reverts ,  
I am not sure whether   Reth tokens  could be burned  or not ,   I think it's reverting , I used hardhat console.log() inside receive function of  
saFETH contract  which gives  this it tranfers ETHER into safETH contract then reverts 

```js
ETHER RECeived : 331157191561750609 
ETHER RECeived : 332871276653896873 
ETHER RECeived : 332897725842115751 
```
``
,   This is reverting but  there could be a  vulnerability 
```js
 CALL rETH(0xae78736cd615f374d3085123a210448e74fc6393).burn(_rethAmount: 311439063843440753)
```




