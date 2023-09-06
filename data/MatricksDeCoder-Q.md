 ### NC1 - Add interfaces to own folder 

[IBondingManager.sol in /bonding/libraries folder](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol)
[IBondingVotes.sol in /bonding/libraries folder](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingVotes.sol)
[IVotes.sol in /treasury](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/IVotes.sol)

**Impact**-> It affects the code  maintainability, readability, organization, auditability and cleanliness of folders and structure 

**Recommendation** -> It is recommended add all interfaces into their own interface folder for all interfaces or create separate interfaces folders within the contract folders where interface is used 
e.g /bonding/libraries/interfaces/IBondingVotes.sol
e.g /treasury/interfaces/IBondingVotes.sol


### NC2 - Not using named imports 

Imports just get entire files e.g 
```solidity 
import "../ManagerProxyTarget.sol";
import "./IBondingManager.sol";
```
[BondingManager.sol lines 4-15](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L4C1-L15C30)
[BondingVotes.sol lines 7-14](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingVotes.sol#L7)
[EarningsPoolLIP36.sol
 lines 4,5](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/libraries/EarningsPoolLIP36.sol#L4)
[SortedArrays.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/libraries/SortedArrays.sol#L4)
[GovernorCountingOverridable.sol lines 8-14](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L8)
[LivepeerGovernor.sol lines 11-18](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/LivepeerGovernor.sol#L11)

**Impact**-> it affects the code organization, readability, interpretability, maintainability, auditability and best practises 

**Recommendation** -> It is recommended to import as named imports as in the examples below 
```solidity
import {ManagerProxyTarget} from "../ManagerProxyTarget.sol";
```

### NC3 - Inconsistent use of underscore _ preceding function arguments

It appears for majority contracts all function parameters make use of _ undescore format e.g
```solidity 
function delegatorCumulativeStakeAt(BondingCheckpoint storage bond, uint256 _round)
```
As in above _round expect for where taking User defined type as in BondingCheckpoint storage bond

However the following functions violate this pattern, best practise of using _ undescore

[SortedArrays.sol... function pushSorted(uint256[] storage array, uint256 val)](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/libraries/SortedArrays.sol#L64C5-L64C62) (val and array) when function in same contract uses underscore format see below 

[SortedArrays.sol ...function findLowerBound(uint256[] storage _array, uint256 _val) internal view returns (uint256) {](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/libraries/SortedArrays.sol#L28C4-L28C102)

[Treasury.sol line 16 ...function initialize( uint256 minDelay, address[] memory proposers, address[] memory executors, address admin) external initializer {](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/Treasury.sol#L16C1-L16C1) 
minDelay and admin should be _minDelay, _admin

[LivepeerGovernor.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L11) does no apply _ underscores for function aparamters/arguments at all unlike all the other contracts 

**Impact**-> it affects the code interpretability, maintainability, readability and consistency

**Recommendation** -> It is recommended to apply consistent usage of underscore _ to all function arguments

