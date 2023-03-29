## [L-01] Users can't stake and unstake if owner added derivative contract with inappropriate interface.
SC: SafEth.sol

The core of lack is lying in function `addDerivative()` where any address can be added as vault.
If in new vault has not implemented IDerivative interface or even uncorrect realization of functions - user's can't stake or stake their funds.
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
        //SafEthProxy amd SafETH deploy;
        safEthSC = new SafEth();
        safERC1967ProxySC = new ERC1967Proxy(address(safEthSC),abi.encodeWithSelector(
                    safEthSC.initialize.selector,
                    "TokenName",
                    "TSBL"
                ));

        SafEthProxy = SafEth(payable(address(safERC1967ProxySC)));

        //RethProxy amd Reth deploy;
        rethEthSC = new Reth();
        rethERC1967ProxySC = new ERC1967Proxy(address(rethEthSC),abi.encodeWithSelector(
                    rethEthSC.initialize.selector,address(SafEthProxy)));

        RethProxy = Reth(payable(address(rethERC1967ProxySC)));

        //SfrxEthProxy amd SfrxEth deploy;
        SfrxEthSC = new SfrxEth();
        sfrxEthERC1967ProxySC = new ERC1967Proxy(address(SfrxEthSC),abi.encodeWithSelector(
                    SfrxEthSC.initialize.selector,address(SafEthProxy)));

        SfrxEthProxy = SfrxEth(payable(address(sfrxEthERC1967ProxySC)));

         //WstEthProxy amd WstEth deploy;
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


## [L-02] Owner can steal all funds.
SC: all

Despite on fact there is no direct function for transfer to owner or other destination there is backdoor in this implementation.
1) Owner set weights for every derivative contract to `0`;
2) Owner add new derivative with weight `100` with implemented functions `deposit()` but custom logic.
3) Owner call `rebalanceToWeights`. 
4) All funds from vault are going to new custom derivativeContract.

## Recommended Mitigation Steps
-Owner should be at least multisig(better dao for adding new derivatives);
