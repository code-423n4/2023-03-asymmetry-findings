### Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G-01] | Multiple accesses of a storage variable should use a local variable cache | 1 |  - |
| [G-02] | Emit memory value instead of state variable | 4 |  388 |
| [G-03] | Sort solidity operations using short-circuit mode | 43 | at least 4171  |
| [G-04] | Functions guaranteed to revert when called by normal users can be marked payable | 17 |  357 |
| [G-05] | Refactor functions `adjustWeight` and `addDerivative` | 2 |  - |
| [G-06] | Shift checks before declaration for possible gas savings | 1 |  - |
| [G-07] | Consider declaring stack variables outside loop to save gas | 1 |  ~6 gas per loop |

| Total Found Issues | 7 |
|:--:|:--:|

### [G-01] Multiple accesses of a storage variable should use a local variable cache

### Cache `STETH_TOKEN` in local variable
```solidity
2 results - 1 file

/WstEth.sol
56:    function withdraw(uint256 _amount) external onlyOwner 
58:        uint256 stEthBal = IERC20(STETH_TOKEN).balanceOf(address(this));
59:        IERC20(STETH_TOKEN).approve(LIDO_CRV_POOL, stEthBal);
```

### Cache `LIDO_CRV_POOL` in local variable
```solidity
2 results - 1 file

/WstEth.sol
56:    function withdraw(uint256 _amount) external onlyOwner 
59:        IERC20(STETH_TOKEN).approve(LIDO_CRV_POOL, stEthBal);
61:        IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);
```

### Cache `WST_ETH` in local variable
```solidity
3 results - 1 file

/WstEth.sol
73:    function deposit() external payable onlyOwner returns (uint256)
74:        uint256 wstEthBalancePre = IWStETH(WST_ETH).balanceOf(address(this));
76:        (bool sent, ) = WST_ETH.call{value: msg.value}("");
78:        uint256 wstEthBalancePost = IWStETH(WST_ETH).balanceOf(address(this));
```

### Cache `FRX_ETH_ADDRESS` in local variable
```solidity
/SfrxEth.sol
2 results - 1 file

60:    function withdraw(uint256 _amount) external onlyOwner
66:        uint256 frxEthBalance = IERC20(FRX_ETH_ADDRESS).balanceOf(
67:            address(this)
68:        );
69:        IsFrxEth(FRX_ETH_ADDRESS).approve(
70:            FRX_ETH_CRV_POOL_ADDRESS,
71:            frxEthBalance
72:        );
```

### Cache `FRX_ETH_CRV_POOL_ADDRESS` in variable
```solidity
2 results - 1 file

/SfrxEth.sol
60:    function withdraw(uint256 _amount) external onlyOwner
69:        IsFrxEth(FRX_ETH_ADDRESS).approve(
70:            FRX_ETH_CRV_POOL_ADDRESS,
71:            frxEthBalance
72:        );

77:        IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).exchange(
78:            1,
79:            0,
80:            frxEthBalance,
81:            minOut
82:        );
````

### Cache `SFRX_ETH_ADDRESS` in local variable
```solidity
2 results - 1 file

/SfrxEth.sol
 94:    function deposit() external payable onlyOwner returns (uint256)
 98:        uint256 sfrxBalancePre = IERC20(SFRX_ETH_ADDRESS).balanceOf(
 99:            address(this)
100:        );

102:        uint256 sfrxBalancePost = IERC20(SFRX_ETH_ADDRESS).balanceOf(
103:            address(this)
104:        );
```

### Cache `totalWeight` within for loop in local variable
```solidity
2 results - 1 file

/SafEth.sol
63:    function stake() external payable
84:        for (uint i = 0; i < derivativeCount; i++) {
85:            uint256 weight = weights[i];
86:            IDerivative derivative = derivatives[i];
87:            if (weight == 0) continue;
88:            uint256 ethAmount = (msg.value * weight) / totalWeight;

138:    function rebalanceToWeights() external onlyOwner
147:        for (uint i = 0; i < derivativeCount; i++) {
148:            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
149:            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
150:                totalWeight;
```

### Cache `UNISWAP_ROUTER` in local variable
```solidity
2 results - 1 file

/Reth.sol
 83:    function swapExactInputSingleHop
 90:        IERC20(_tokenIn).approve(UNISWAP_ROUTER, _amountIn);
101:        amountOut = ISwapRouter(UNISWAP_ROUTER).exactInputSingle(params);
```

### Cache `ROCKET_STORAGE_ADDRESS` in local variable
```solidity
2 results - 1 file

/Reth.sol
120:    function poolCanDeposit(uint256 _amount) private view returns (bool)
122:            ROCKET_STORAGE_ADDRESS
133:            ROCKET_STORAGE_ADDRESS
```

### Cache `derivativeCount` in local storage variable
```solidity
9 results - 1 file

/SafEth.sol
63:    function stake() external payable
71:        for (uint i = 0; i < derivativeCount; i++)
84:        for (uint i = 0; i < derivativeCount; i++)


108:    function unstake(uint256 _safEthAmount) external
113:        for (uint256 i = 0; i < derivativeCount; i++)

138:    function rebalanceToWeights() external onlyOwner
140:        for (uint i = 0; i < derivativeCount; i++)
147:        for (uint i = 0; i < derivativeCount; i++)

165:    function adjustWeight(
171:        for (uint256 i = 0; i < derivativeCount; i++)

182:    function addDerivative(
186:        derivatives[derivativeCount] = IDerivative(_contractAddress);
187:        weights[derivativeCount] = _weight;
191:        for (uint256 i = 0; i < derivativeCount; i++)
```

### Cache `derivatives` in local storage variable
```solidity
10 results - 1 file

/SafEth.sol
63:    function stake() external payable 
71:        for (uint i = 0; i < derivativeCount; i++)
72:            underlyingValue +=
73:                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74:                    derivatives[i].balance()) /
84:        for (uint i = 0; i < derivativeCount; i++) {
85:            uint256 weight = weights[i];
86:            IDerivative derivative = derivatives[i];

108:    function unstake(uint256 _safEthAmount) external
115:            uint256 derivativeAmount = (derivatives[i].balance() *
116:                _safEthAmount) / safEthTotalSupply;
117:            if (derivativeAmount == 0) continue; // if derivative empty ignore
118:            derivatives[i].withdraw(derivativeAmount);

138:    function rebalanceToWeights() external onlyOwner
140:        for (uint i = 0; i < derivativeCount; i++) {
141:            if (derivatives[i].balance() > 0)
142:                derivatives[i].withdraw(derivatives[i].balance());
143:        }
147:        for (uint i = 0; i < derivativeCount; i++) {
148:            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
149:            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
150:                totalWeight;
151:            // Price will change due to slippage
152:            derivatives[i].deposit{value: ethAmount}();
153:        }
```
### Cache `weights` in local storage variable
```solidity
5 results - 1 file

/SafeEth.sol
63:    function stake() external payable
84:        for (uint i = 0; i < derivativeCount; i++) {
85:            uint256 weight = weights[i];

138:    function rebalanceToWeights() external onlyOwner
147:        for (uint i = 0; i < derivativeCount; i++) {
148:            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
149:            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
150:                totalWeight;

165:    function adjustWeight
171:        for (uint256 i = 0; i < derivativeCount; i++)
172:            localTotalWeight += weights[i];

182:    function addDerivative
191:        for (uint256 i = 0; i < derivativeCount; i++)
192:            localTotalWeight += weights[i];
```

Description:
SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.
***

### [G-02] Emit memory value instead of state variable

```solidity
4 results - 1 file

/SafEth.sol
214:    function setMinAmount(uint256 _minAmount) external onlyOwner {
215:        minAmount = _minAmount;
216:        emit ChangeMinAmount(minAmount);
217:    }

223:    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
224:        maxAmount = _maxAmount;
225:        emit ChangeMaxAmount(maxAmount);
226:    }

232:    function setPauseStaking(bool _pause) external onlyOwner {
233:        pauseStaking = _pause;
234:        emit StakingPaused(pauseStaking);
235:    }

241:    function setPauseUnstaking(bool _pause) external onlyOwner {
242:        pauseUnstaking = _pause;
243:        emit UnstakingPaused(pauseUnstaking);
244:    }
```

The above instances can be refactored to emit the argument inputted stored in memory instead of emitting the storage variable. This saves gas as it replaces a SLOADs (Gwarmaccess, 100 gas) with a MLOAD (3 gas)

### [G-03] Sort solidity operations using short-circuit mode
```solidity
1 result - 1 file

/SafEth.sol
138:    function rebalanceToWeights() external onlyOwner
148:            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
```

We can sequence `OR/AND (|| , &)` operators in solidity as it applies the common short circuiting rule. 

This means that in the expression `f(x) || g(y)`, if `f(x)` evaluates to true, `g(y)` will not be evaluated even if it may have side-effects. 

So setting less costly function to “f(x)” and setting costly function to “g(x)” is more gas efficient.

### [G-04] Functions guaranteed to revert when called by normal users can be marked payable
```solidity
17 results - 4 files

/WstEth.sol
48: function setMaxSlippage(uint256 _slippage) external onlyOwner
56: function withdraw(uint256 _amount) external onlyOwner
73: function deposit() external payable onlyOwner returns (uint256)

/SfrxEth.sol
51: function setMaxSlippage(uint256 _slippage) external onlyOwner
60: function withdraw(uint256 _amount) external onlyOwner
94: function deposit() external payable onlyOwner returns (uint256)

/SafEth.sol
138: function rebalanceToWeights() external onlyOwner 

165:    function adjustWeight(
166:        uint256 167:_derivativeIndex,
168:        uint256 _weight
169:    ) external onlyOwner

182:    function addDerivative(
183:        address _contractAddress,
184:        uint256 _weight
185:    ) external onlyOwner

202:    function setMaxSlippage(
203:        uint _derivativeIndex,
204:        uint _slippage
205:    ) external onlyOwner

214: function setMinAmount(uint256 _minAmount) external onlyOwner 
223: function setMaxAmount(uint256 _maxAmount) external onlyOwner
232: function setPauseStaking(bool _pause) external onlyOwner
241: function setPauseUnstaking(bool _pause) external onlyOwner

/Reth.sol
58: function setMaxSlippage(uint256 _slippage) external onlyOwner
107: function withdraw(uint256 amount) external onlyOwner
156: function deposit() external payable onlyOwner returns (uint256)
```

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

Consider marking the functions as payable. However, it is valid if protocol decides against it due to confusion, readability and possibility of receiving funds.

## [G-05] Refactor functions `adjustWeight` and `addDerivative`
```solidity
2 results - 2 files

/SafEth.sol
165:    function adjustWeight(
166:        uint256 _derivativeIndex,
167:        uint256 _weight
168:    ) external onlyOwner {
169:        weights[_derivativeIndex] = _weight;
170:        uint256 localTotalWeight = 0;
171:        for (uint256 i = 0; i < derivativeCount; i++)
172:            localTotalWeight += weights[i];
173:        totalWeight = localTotalWeight;
174:        emit WeightChange(_derivativeIndex, _weight);
175:    }

182:    function addDerivative(
183:        address _contractAddress,
184:        uint256 _weight
185:    ) external onlyOwner {
186:        derivatives[derivativeCount] = IDerivative(_contractAddress);
187:        weights[derivativeCount] = _weight;
188:        derivativeCount++;
189:
190:        uint256 localTotalWeight = 0;
191:        for (uint256 i = 0; i < derivativeCount; i++)
192:            localTotalWeight += weights[i];
193:        totalWeight = localTotalWeight;
194:        emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
195:    }
```

For the above functions `adjustWeight` and `function addDerivative`, there is no need to loop through the whole `weights` mapping to update `totalWeight`. We can simply update the `totalWeight` state variable by adding the new `_weight` to the `totalWeight` variable. This can also have significant gas savings by removing the for loops.

```solidity
function adjustWeight(
    uint256 _derivativeIndex,
    uint256 _weight
) external onlyOwner {
    weights[_derivativeIndex] = _weight;
    totalWeight = totalWeight + _weight;
    emit WeightChange(_derivativeIndex, _weight);
}


function addDerivative(
    address _contractAddress,
    uint256 _weight
) external onlyOwner {
    derivatives[derivativeCount] = IDerivative(_contractAddress);
    weights[derivativeCount] = _weight;
    derivativeCount++;

    totalWeight = totalWeight + _weight;
    emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
}
```

## [G-06] Shift checks before declaration for possible gas savings

```solidity
1 result - 1 file

/SafEth.sol
63:    function stake() external payable
        ...
84:        for (uint i = 0; i < derivativeCount; i++) {
85:            uint256 weight = weights[i];
86:            IDerivative derivative = derivatives[i];
87:            if (weight == 0) continue;
88:            uint256 ethAmount = (msg.value * weight) / totalWeight;
```

In the above instance, shift the check of
`if (weight == 0) continue;` before declaring `IDerivative derivative` so that the for loop will properly skip `weight` with value of 0 without wasting gas by unnecessarily declaring the derivative contract for further computation.

```solidity
function stake() external payable{
    ...
    for (uint i = 0; i < derivativeCount; i++) {
        uint256 weight = weights[i];
        if (weight == 0) continue;
        IDerivative derivative = derivatives[i];       
```

## [G-07] Consider declaring stack variables outside loop to save gas
```solidity
7 results - 1 file

/SafEth.sol
63:     function stake() external payable
84:        for (uint i = 0; i < derivativeCount; i++) {
85:            uint256 weight = weights[i];
86:            IDerivative derivative = derivatives[i];
88:            uint256 ethAmount = (msg.value * weight) / totalWeight;
91:            uint256 depositAmount = derivative.deposit{value: ethAmount}();
92:            uint derivativeReceivedEthValue

108:    function unstake(uint256 _safEthAmount) external
113:        for (uint256 i = 0; i < derivativeCount; i++)
115:            uint256 derivativeAmount

138:     function rebalanceToWeights() external onlyOwner
147:        for (uint i = 0; i < derivativeCount; i++)
149:            uint256 ethAmount
```

Description:
Consider initializing the stack variables before the loop to avoid reinitialization on every loop to save gas. Saves  around [~6 gas per loop](https://ethereum.stackexchange.com/questions/118754/is-it-more-gas-efficient-to-declare-variable-inside-or-outside-of-a-for-or-while)

