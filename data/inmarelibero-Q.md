\==============================================================================================================
\# 1
\==============================================================================================================

# Abstract
At line #194 the wrong `index` is included in the event.

```
File: contracts/SafEth/SafEth.sol
182:     function addDerivative(
183:         address _contractAddress,
184:         uint256 _weight
185:     ) external onlyOwner {
186:         derivatives[derivativeCount] = IDerivative(_contractAddress);
187:         weights[derivativeCount] = _weight;
188:         derivativeCount++;
189:
190:         uint256 localTotalWeight = 0;
191:         for (uint256 i = 0; i < derivativeCount; i++)
192:             localTotalWeight += weights[i];
193:         totalWeight = localTotalWeight;
194:         emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
195:     }

```

For instance, when adding the first `derivative`, the third parameter `index` to be passed to the `DerivativeAdded` event should be 0.

```
event DerivativeAdded(
    address indexed contractAddress,
    uint weight,
    uint index
);
```

When `188: derivativeCount++; ` is called before `194: emit DerivativeAdded(_contractAddress, _weight, derivativeCount);`, the count is incremented and then `index` will be 1 instead of 0.


# Patch

Increment `derivativeCount` after emitting event.

```
emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
derivativeCount++;
```

\==============================================================================================================
\# 2
\==============================================================================================================

# Abstract

Avoid duplicated fragment of code when updating `totalWeight`

Same logic is repeated here https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L170 and here https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L170

Avoid duplicating code for better bugfixing and less prone to errors.

# Patch

Create a private method `_adjustWeight()` called both by `adjustWeight()` and `addDerivative()`


```
File: contracts/SafEth/SafEth.sol
157:     /**
158:         @notice - Adds new derivative to the index fund
159:         @dev - Weights are only in regards to each other, total weight changes with this function
160:         @dev - If you want exact weights either do the math off chain or reset all existing derivates to the weights you want
161:         @dev - Weights are approximate as it will slowly change as people stake
162:         @param _derivativeIndex - index of the derivative you want to update the weight
163:         @param _weight - new weight for this derivative.
164:     */
165:     function adjustWeight(
166:         uint256 _derivativeIndex,
167:         uint256 _weight
168:     ) external onlyOwner {
169:         _adjustWeight(_derivativeIndex, _weight);
170:         emit WeightChange(_derivativeIndex, _weight);
171:     }
172:
173:     /**
174:         @notice - Adds new derivative to the index fund
175:         @dev - Weights are only in regards to each other, total weight changes with this function
176:         @dev - If you want exact weights either do the math off chain or reset all existing derivates to the weights you want
177:         @dev - Weights are approximate as it will slowly change as people stake
178:         @param _derivativeIndex - index of the derivative you want to update the weight
179:         @param _weight - new weight for this derivative.
180:     */
181:     function _adjustWeight(
182:         uint256 _derivativeIndex,
183:         uint256 _weight
184:     ) private onlyOwner {
185:         weights[_derivativeIndex] = _weight;
186:         uint256 localTotalWeight = 0;
187:         for (uint256 i = 0; i < derivativeCount; i++)
188:             localTotalWeight += weights[i];
189:         totalWeight = localTotalWeight;
190:     }
191:
192:     /**
193:         @notice - Adds new derivative to the index fund
194:         @param _contractAddress - Address of the derivative contract launched by AF
195:         @param _weight - new weight for this derivative.
196:     */
197:     function addDerivative(
198:         address _contractAddress,
199:         uint256 _weight
200:     ) external onlyOwner {
201:         uint256 _derivativeIndex = derivativeCount;
202:
203:         derivatives[_derivativeIndex] = IDerivative(_contractAddress);
204:         derivativeCount++;
205:         _adjustWeight(_derivativeIndex, _weight);
206:
207:         emit DerivativeAdded(_contractAddress, _weight, _derivativeIndex);
208:     }

```