## [L-01] Users can't stake and unstake if owner added derivative contract with inappropriate interface.
SC: SafEth.sol

The core of lack is lying in function `addDerivative()` where any address can be added as vault.
If in new vault has not implemented IDerivative interface or even uncorrect realization of functions - user's can't stake or stake their funds. Also there is no function to delete derivative contract.
![](https://i.imgur.com/YUZkIb1.png)
 
## Proof of Concept
Foundry test:
`forge test --fork-url mainnet -vvv`
```solidity
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "../../src/SafEth/SafEth.sol";
import "../../src/SafEth/derivatives/Reth.sol";
import "../../src/SafEth/derivatives/SfrxEth.sol";
import "../../src/SafEth/derivatives/WstEth.sol";
import "@openzeppelin/contracts/proxy/ERC1967/ERC1967Proxy.sol";


contract SafEthAudit is Test {

    SafEth public safEthSC;
    SafEth public SafEthProxy;
    ERC1967Proxy public safERC1967ProxySC;

    Reth public rethEthSC;
    Reth public RethProxy;
    ERC1967Proxy public rethERC1967ProxySC;

    SfrxEth public SfrxEthSC;
    SfrxEth public SfrxEthProxy;
    ERC1967Proxy public sfrxEthERC1967ProxySC;

    WstEth public WstEthSC;
    WstEth public WstEthProxy;
    ERC1967Proxy public wstEthERC1967ProxySC;

    address eoa = vm.addr(123);
    address user1 = vm.addr(12347);
    address attacker = vm.addr(1234);
    

    function setUp() public {

        vm.deal(eoa, 100 ether);
        vm.deal(user1, 100 ether);
        vm.deal(attacker, 100000 ether);

        vm.startPrank(eoa);
        //SafEthProxy and SafETH deploy;
        safEthSC = new SafEth();
        safERC1967ProxySC = new ERC1967Proxy(address(safEthSC),abi.encodeWithSelector(
                    safEthSC.initialize.selector,
                    "TokenName",
                    "TSBL"
                ));

        SafEthProxy = SafEth(payable(address(safERC1967ProxySC)));

        //RethProxy and Reth deploy;
        rethEthSC = new Reth();
        rethERC1967ProxySC = new ERC1967Proxy(address(rethEthSC),abi.encodeWithSelector(
                    rethEthSC.initialize.selector,address(SafEthProxy)));

        RethProxy = Reth(payable(address(rethERC1967ProxySC)));

        //SfrxEthProxy and SfrxEth deploy;
        SfrxEthSC = new SfrxEth();
        sfrxEthERC1967ProxySC = new ERC1967Proxy(address(SfrxEthSC),abi.encodeWithSelector(
                    SfrxEthSC.initialize.selector,address(SafEthProxy)));

        SfrxEthProxy = SfrxEth(payable(address(sfrxEthERC1967ProxySC)));

         //WstEthProxy and WstEth deploy;
        WstEthSC = new WstEth();
        wstEthERC1967ProxySC = new ERC1967Proxy(address(WstEthSC),abi.encodeWithSelector(
                    WstEthSC.initialize.selector,address(SafEthProxy)));

        WstEthProxy = WstEth(payable(address(wstEthERC1967ProxySC)));

        console.log("eoa address:                            ",address(eoa));
        console.log("SafEthProxy address:                    ",address(SafEthProxy));
        console.log("safEthSC address impl:                  ",address(safEthSC));
        console.log("==============================================================");
        console.log("RethProxy address:                      ",address(RethProxy));
        console.log("rethEthSC address impl:                 ",address(rethEthSC));
        console.log("==============================================================");
        console.log("SfrxEthProxy address:                   ",address(SfrxEthProxy));
        console.log("SfrxEthSC address impl:                 ",address(SfrxEthSC));
        console.log("==============================================================");
        console.log("WstEthProxy address:                    ",address(WstEthProxy));
        console.log("WstEthSC address impl:                  ",address(WstEthSC));

        //Start config

        SafEthProxy.addDerivative(address(RethProxy),100);
        SafEthProxy.addDerivative(address(SfrxEthProxy),100);
        SafEthProxy.addDerivative(address(WstEthProxy),100);
        
        SafEthProxy.setMaxSlippage(0,1e16);  
        SafEthProxy.setMaxSlippage(1,1e16);
        SafEthProxy.setMaxSlippage(2,1e16);

        SafEthProxy.setMinAmount(0);
        SafEthProxy.setMaxAmount(200e18);
        vm.stopPrank();
    }
    
    uint256 maxSlip = 1e16;
  
    function testStake() public {
        vm.startPrank(user1);
      
        SafEthProxy.stake{value: 1 ether}();
        uint256 balanceLP = SafEthProxy.balanceOf(user1);
        vm.stopPrank();

        vm.prank(eoa);
        SafEthProxy.addDerivative(address(msg.sender),100);
        
        vm.startPrank(user1);
        SafEthProxy.unstake(balanceLP);
    }
}
```
![](https://i.imgur.com/HPo2HxR.png);


## Recommended Mitigation Steps
Add in function `addDerivative` checking process:
1) `require((bool success,) = derivative.call(abi.encodeWithSignature("deposit()")),"Incorrect Interface");`
2) `require((bool success,) = derivative.call(abi.encodeWithSignature("withdraw()")),"Incorrect Interface");`
3) `require (success)`


## [L-02] Absence of slippage control could lead to lost user funds in some scenarios or return unexpected variable.
## Impact
SC: All in scope.
Despite on fact that `maxSlippage` is being set during `initialize()` process in every derivative contract, at `SafEth.sol` there is function `setMaxSlippage()` that allows to set any slippage from 0 to uint256.max. There is no `require(maxSlippage < 1e17)` by default. 

So there are some scenarios when user funds could be lost. 
1 . If owner sets `maxSlippage` more than `1e18` in `WstEth.sol` the transaction will be reverted. (due to the no sign in uint);
2 . Despite on fact that `setMinAmount` is setting during `initialize()` process in every derivative contract, at `SafEth.sol` there is function `setMinAmount()` which allows to set any amount with no requirements).  So user deposits 10 wei. Slippage currently 1% (1e16). Actual `minOut` in `withdraw()` function will be 9 wei. (9.99 round down due to the EVM principles.) What at fact 10% slippage. Or other example: user deposits 101 wei, where slippage is set to 10%. Actual slippage = 10.1, but EVM slippage = 10. User could lost more than his script configured.

Such no-value loses can actually break logic if user use scripts or other smart contract which relies to actual smart contract code.

## Proof of Concept

Short test with another one example:
```solidity
uint256 maxSlip = 1e16;

    function testFuzz_Slippage(uint256 ethDep) public {
        vm.assume(ethDep > 1 ether && ethDep < 200 ether);
        console.log("ethDep:             ",ethDep);
        console.log("maxSlip:            ",maxSlip);
        uint256 minOutSC = (ethDep * (10 ** 18 - maxSlip)) / 10 ** 18;
        uint256 minOutReal = ethDep - 1e16 * ethDep / 1e18;
        uint256 actualSlippage = ethDep - minOutSC;
        
        console.log("Slippace calculated in current SC:              ",1e16);
        console.log("Slippace calculated appropriatly w/o roundring: ",actualSlippage);
        
        assertEq(1e16,actualSlippage);
    
    }
```
Full test with deployment process.
```solidity
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "../../src/SafEth/SafEth.sol";
import "../../src/SafEth/derivatives/Reth.sol";
import "../../src/SafEth/derivatives/SfrxEth.sol";
import "../../src/SafEth/derivatives/WstEth.sol";
import "@openzeppelin/contracts/proxy/ERC1967/ERC1967Proxy.sol";


contract SafEthAudit is Test {

    SafEth public safEthSC;
    SafEth public SafEthProxy;
    ERC1967Proxy public safERC1967ProxySC;

    Reth public rethEthSC;
    Reth public RethProxy;
    ERC1967Proxy public rethERC1967ProxySC;

    SfrxEth public SfrxEthSC;
    SfrxEth public SfrxEthProxy;
    ERC1967Proxy public sfrxEthERC1967ProxySC;

    WstEth public WstEthSC;
    WstEth public WstEthProxy;
    ERC1967Proxy public wstEthERC1967ProxySC;

    address eoa = vm.addr(123);

    address user1 = vm.addr(12347);
    address attacker = vm.addr(1234);

    function setUp() public {

        vm.startPrank(eoa);
        //SafEthProxy and SafETH deploy;
        safEthSC = new SafEth();
        safERC1967ProxySC = new ERC1967Proxy(address(safEthSC),abi.encodeWithSelector(
                    safEthSC.initialize.selector,
                    "TokenName",
                    "TSBL"
                ));

        SafEthProxy = SafEth(payable(address(safERC1967ProxySC)));

        //RethProxy and Reth deploy;
        rethEthSC = new Reth();
        rethERC1967ProxySC = new ERC1967Proxy(address(rethEthSC),abi.encodeWithSelector(
                    rethEthSC.initialize.selector,address(SafEthProxy)));

        RethProxy = Reth(payable(address(rethERC1967ProxySC)));

        //SfrxEthProxy and SfrxEth deploy;
        SfrxEthSC = new SfrxEth();
        sfrxEthERC1967ProxySC = new ERC1967Proxy(address(SfrxEthSC),abi.encodeWithSelector(
                    SfrxEthSC.initialize.selector,address(SafEthProxy)));

        SfrxEthProxy = SfrxEth(payable(address(sfrxEthERC1967ProxySC)));

         //WstEthProxy and WstEth deploy;
        WstEthSC = new WstEth();
        wstEthERC1967ProxySC = new ERC1967Proxy(address(WstEthSC),abi.encodeWithSelector(
                    WstEthSC.initialize.selector,address(SafEthProxy)));

        WstEthProxy = WstEth(payable(address(wstEthERC1967ProxySC)));

        console.log("eoa address:                            ",address(eoa));
        console.log("SafEthProxy address:                    ",address(SafEthProxy));
        console.log("safEthSC address impl:                  ",address(safEthSC));
        console.log("==============================================================");
        console.log("RethProxy address:                      ",address(RethProxy));
        console.log("rethEthSC address impl:                 ",address(rethEthSC));
        console.log("==============================================================");
        console.log("SfrxEthProxy address:                   ",address(SfrxEthProxy));
        console.log("SfrxEthSC address impl:                 ",address(SfrxEthSC));
        console.log("==============================================================");
        console.log("WstEthProxy address:                    ",address(WstEthProxy));
        console.log("WstEthSC address impl:                  ",address(WstEthSC));

        //Start config

        SafEthProxy.addDerivative(address(RethProxy),100);
        SafEthProxy.addDerivative(address(SfrxEthProxy),100);
        SafEthProxy.addDerivative(address(WstEthProxy),100);

        SafEthProxy.setMaxSlippage(0,10);  
        SafEthProxy.setMaxSlippage(1,10);
        SafEthProxy.setMaxSlippage(2,10);

        SafEthProxy.setMinAmount(5e17);
        SafEthProxy.setMaxAmount(200e18);
          
    }
    uint256 maxSlip = 1e16;

    function testFuzz_Slippage(uint256 ethDep) public {
        vm.assume(ethDep > 1 ether && ethDep < 200 ether);
        console.log("ethDep:             ",ethDep);
        console.log("maxSlip:            ",maxSlip);
        uint256 minOutSC = (ethDep * (10 ** 18 - maxSlip)) / 10 ** 18;
        uint256 minOutReal = ethDep - 1e16 * ethDep / 1e18;
        uint256 actualSlippage = ethDep - minOutSC;
        
        console.log("Slippace calculated in current SC:              ",1e16);
        console.log("Slippace calculated appropriatly w/o roundring: ",actualSlippage);
        
        assertEq(1e16,actualSlippage);
    
    }
}
```
As we can see difference:
![](https://i.imgur.com/e6UFWuT.png)

# Tools Used
Manual review

# Recommended Mitigation Steps
Use `require(x % == 0)`   definition or mantissa libraries.


## [L-03] Owner can steal all funds.
SC: all

Despite on fact there is no direct function for transfer to owner or other destination there is backdoor in this implementation.
-Owner set weights for every derivative contract to `0`;
-Owner add new derivative with weight `100` with implemented functions `deposit()` but custom logic.
-Owner call `rebalanceToWeights`. 
-All funds from vault are going to new custom derivativeContract.

## Recommended Mitigation Steps
Owner have to be at least multisig(better DAO for adding new derivatives);
