# 1: FLOATING PRAGMA

Vulnerability details

## Context

Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the floating pragma, i.e. by not using ^ in pragma solidity ^0.8.13, ensures that contracts do not accidentally get deployed using an older compiler version with unfixed bugs.

For reference, see https://swcregistry.io/docs/SWC-103

## Proof of Concept

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L2

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L2

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L2

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L2

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Remove ^ in “pragma solidity ^0.8.13” and change it to “pragma solidity 0.8.13” to be consistent with the rest of the contracts.


# 2: CREATE YOUR OWN IMPORT NAMES INSTEAD OF USING THE REGULAR ONES

Vulnerability details

## Context:

For better readability, you should name the imports instead of using the regular ones.

## Proof of Concept

> ***Instances - All contracts in scope.***

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L4-L8

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L4-L9

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L4-L11

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L4-L15

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

import {contract1 , contract2} from "filename.sol";

#### Example:

import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";

# 3: IMPORTS CAN BE GROUPED TOGETHER

Vulnerability details

## Context:

Imports can be grouped together

## Proof of Concept

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L4-L8

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L4-L9

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L4-L11

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L4-L15

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Consider importing OZ first, then all interfaces, then all utils if available.

# 4: INTERCHANGEABLE USAGE OF UINT AND UINT256

Vulnerability details

## Context:

Interchangeable usage of uint and uint256. Below are instances where uint was used rather than uint256 like in the rest of the code.

## Proof of Concept

> ***File: SafEth.sol***

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L26-L28

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L31-L32

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L71

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L84

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L92

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L140

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L147

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L203-L204

> ***File: Reth.sol***

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L171

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Consider using only one approach throughout the codebase, e.g. only uint or only uint256.

# 5: GENERATE PERFECT CODE HEADERS EVERY TIME

Vulnerability details

## Impact:

Generate perfect code headers every time

For reference, see https://github.com/transmissions11/headers

## Proof of Concept

/*//////////////////////////////////////////////////////////////
                                                                 TESTING 123
//////////////////////////////////////////////////////////////*/

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

We recommend using headers for Solidity code layout and readability.


# 6: USE bytes.concat() INSTEAD OF abi.encodePacked()

Vulnerability details

## Context:

Use bytes.concat() instead of abi.encodePacked()

## Proof of Concept

Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled

#### 1 Result - 6 Instances

> ***File: Reth.sol***

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L70

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L125

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L136

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L162

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L191

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L233

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked().

# 7: MOVE REQUIRE/VALIDATION STATEMENTS TO THE TOP OF THE FUNCTION WHEN VALIDATING INPUT PARAMETERS

Vulnerability details

## Context:

Move require/validation statements to the top of the function when validating input parameters

## Proof of Concept

> ***File: WstEth.sol***

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L66

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/WstEth.sol#L77

> ***File: SfrxEth.sol***

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/SfrxEth.sol#L87

> ***File: SafEth.sol***

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L127

> ***File: Reth.sol***

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L113

https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/derivatives/Reth.sol#L200

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

> ***File: WstEth.sol***

Consider moving the validation on L66 above the conditional on L57 for withdraw().

Consider moving the validation on L77 above the conditional on L74 for deposit().

> ***File: SfrxEth.sol***

Consider moving the validation on L87 above the conditional on L61 for withdraw().

> ***File: SafEth.sol***

Consider moving the validation on L127 above the conditional on L110 for unstake().

> ***File: Reth.sol***

Consider moving the validation on L113 above the conditional on L108 for withdraw().

Consider moving the validation on L200 above the conditional on L158 for deposit().


# 8: FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE

Vulnerability details

## Context:

Order of Functions; ordering helps readers identify which functions they can call and find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

For reference, see https://docs.soliditylang.org/en/v0.8.17/style-guide.html

## Proof of Concept

> ***Functions should be grouped according to their visibility and ordered:***

-constructor
-receive function (if exists)
-fallback function (if exists)
-external
-public
-internal
-private
-within a grouping, place the view and pure functions last

### Tools Used

Manual Analysis

