1. User is unable withdraw eth if a derivative is added afterwards and adjustWeight() is called
Say there are 2 derivatives with 50/50 adjusted weight
Then user stakes
the another derivative is added and weight adjusted is 33/33/33
then another user stakes.
If the first user tries to unstake it gives error. The user should have been able to withdraw but transaction reverts with `Insufficient rETH balance`

```
 it.only("Check what happens if a derivative is added afterwards", async () => {
        const accounts = await ethers.getSigners();
        const derivativeCount = (await safEthProxy.derivativeCount()).toNumber();
    
        const initialWeight = BigNumber.from("1000000000000000000");
        const initialDeposit = ethers.utils.parseEther("1");
    
        const balanceBefore = await adminAccount.getBalance();
    
        let totalNetworkFee = BigNumber.from(0);
        // set all derivatives to the same weight and stake
        // if there are 3 derivatives this is 33/33/33
        console.log("DERIVARTIVE COUNT::", derivativeCount);
        for (let i = 0; i < derivativeCount; i++) {
          const tx1 = await safEthProxy.adjustWeight(i, initialWeight);
          const mined1 = await tx1.wait();
          const networkFee1 = mined1.gasUsed.mul(mined1.effectiveGasPrice);
          totalNetworkFee = totalNetworkFee.add(networkFee1);
        }
    
        const tx2 = await safEthProxy.stake({ value: initialDeposit });
        const mined2 = await tx2.wait();
        const networkFee2 = mined2.gasUsed.mul(mined2.effectiveGasPrice);
        totalNetworkFee = totalNetworkFee.add(networkFee2);
        let balBefore: any = await safEthProxy.balanceOf(adminAccount.address);
        let rethBalBefore: any = await derivative0.balance();
        let sfrxBalBefore: any = await derivative1.balance();
        console.log("balance before unstake::", balBefore);
        console.log("reth balance before::", rethBalBefore);
        console.log("sfrx balance before::", sfrxBalBefore);

        await safEthProxy.addDerivative(derivative2.address, "1000000000000000000");

        const derivativeCountAfter = (await safEthProxy.derivativeCount()).toNumber();

        console.log("DERIVARTIVE COUNT::", derivativeCountAfter);
        for (let i = 0; i < derivativeCount; i++) {
            const tx3 = await safEthProxy.adjustWeight(i, initialWeight);
            const mined3 = await tx3.wait();
            const networkFee1 = mined3.gasUsed.mul(mined3.effectiveGasPrice);
            totalNetworkFee = totalNetworkFee.add(networkFee1);
        }
    
        const tx4 = await safEthProxy.connect(accounts[3]).stake({ value: initialDeposit });
        const mined4 = await tx4.wait();
        const networkFee4 = mined4.gasUsed.mul(mined4.effectiveGasPrice);
        totalNetworkFee = totalNetworkFee.add(networkFee4);
        const balBeforeAcc3 = await safEthProxy.balanceOf(accounts[3].address);
        console.log("balance of 2nd staker::", balBeforeAcc3);

        let balAfterNewPool: any = await safEthProxy.balanceOf(adminAccount.address);
        let rethBalAfterNewPool: any = await derivative0.balance();
        let sfrxBalAfterNewPool: any = await derivative1.balance();
        let wstPoolBal: any = await derivative2.balance();
        console.log("\n balance before unstake::", balAfterNewPool);
        console.log("reth balance before::", rethBalAfterNewPool);
        console.log("sfrx balance before::", sfrxBalAfterNewPool);
        console.log("wst pool balance::", wstPoolBal);

        const tx5 = await safEthProxy.unstake(
            ethers.utils.parseEther(`${balAfterNewPool}`)
        );
        await tx5.wait();
        let balanceAfter: any = await safEthProxy.balanceOf(adminAccount.address);
        let rethBalAfterUnstake: any = await derivative0.balance();
        let sfrxBalAfterUnstake: any = await derivative1.balance();
        let wstPoolBalAfterUnstake: any = await derivative2.balance();
        console.log("\n balance after unstake::", balanceAfter);
        console.log("reth balance before::", rethBalAfterUnstake);
        console.log("sfrx balance before::", sfrxBalAfterUnstake);
        console.log("wst pool balance::", wstPoolBalAfterUnstake);
    
        balBefore = await safEthProxy.balanceOf(accounts[3].address);
        console.log("balance of acc3 after unstake::", balBefore);
      })
```