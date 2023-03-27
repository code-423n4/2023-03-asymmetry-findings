            State variables used in for loops should be stored in memory rather than reading it directly from storage.
State variables used in for loops should be cached in memory rather than reading it directly from storage ,to read a state variable from storage for every iteration uses the (Sload opcode which cost 200 gas per iteration),but if it is stored in memory (Sload is called once and for the rest of the iterations read from memory which cost only 3 gas per iteration) that saves 197 gas in the remaining iterations.
Variable affected :derivativeCount
Functions affected:Stake,Unstake,rebalanceToweights,adjustWeights,addDerivative.

function save()public {
for(uint i;i<derivativeCount;i++){
    }
	}
function _save()public{// In 10 iterations this function consumed 1000 less gas by storing the 
uint Help= derivativeCount;
function _save()public{
for(uint i;i<Help;i++){
}
}


