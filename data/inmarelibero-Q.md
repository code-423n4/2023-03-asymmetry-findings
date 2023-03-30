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