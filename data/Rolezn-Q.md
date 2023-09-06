## QA Summary<a name="QA Summary">

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Empty `bytes` check is missing | 3 |
| [NC&#x2011;2](#NC&#x2011;2) | File does not contain any NatSpec at all | 1 |
| [NC&#x2011;3](#NC&#x2011;3) | Some functions contain the same exact logic | 2 |
| [NC&#x2011;4](#NC&#x2011;4) | Pausing withdrawals is unfair to the users | 2 |
| [NC&#x2011;5](#NC&#x2011;5) | Redundant inheritance specifier | 1 |
| [NC&#x2011;6](#NC&#x2011;6) | Visibility should be set explicitly rather than defaulting to `internal` | 1 |

Total: 10 contexts over 6 issues



## Non Critical Issues

### <a href="#qa-summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Empty `bytes` check is missing

To avoid mistakenly accepting empty `bytes` as a valid value, consider to check for empty `bytes`. This helps ensure that empty `bytes` are not treated as valid inputs, preventing potential issues.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: LivepeerGovernor.sol

132: function _execute(
        uint256 proposalId,
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        bytes32 descriptionHash
    ) internal override(GovernorUpgradeable, GovernorTimelockControlUpgradeable) {
```

https://github.com/code-423n4/2023-08-livepeer/tree/main/contracts/treasury/LivepeerGovernor.sol#L132


```solidity
File: LivepeerGovernor.sol

142: function _cancel(
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        bytes32 descriptionHash
    ) internal override(GovernorUpgradeable, GovernorTimelockControlUpgradeable) returns (uint256) {
```

https://github.com/code-423n4/2023-08-livepeer/tree/main/contracts/treasury/LivepeerGovernor.sol#L142

```solidity
File: LivepeerGovernor.sol

160: function supportsInterface(bytes4 interfaceId)
        public
        view
        override(GovernorUpgradeable, GovernorTimelockControlUpgradeable)
        returns (bool)
    {
```

https://github.com/code-423n4/2023-08-livepeer/tree/main/contracts/treasury/LivepeerGovernor.sol#L160



</details>



### <a href="#qa-summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> File does not contain any NatSpec at all

#### <ins>Proof Of Concept</ins>

```solidity
File: 

IVotes.sol
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/IVotes.sol




### <a href="#qa-summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Some functions contain the same exact logic

These functions might be a problem if the logic changes before the contract is deployed, as the developer must remember to syncronize the logic between all the function instances.

Consider using a single function instead of duplicating the code, for example by using a `library`, or through inheritance.

#### <ins>Proof Of Concept</ins>


```solidity
File: BondingManager.sol

1639: function roundsManager() internal view returns (IRoundsManager) {
        return IRoundsManager(controller.getContract(keccak256("RoundsManager")));
    }

```

https://github.com/code-423n4/2023-08-livepeer/tree/main/contracts/bonding/BondingManager.sol#L1639

```solidity
File: BondingVotes.sol

546: function roundsManager() internal view returns (IRoundsManager) {
        return IRoundsManager(controller.getContract(keccak256("RoundsManager")));
    }

```

https://github.com/code-423n4/2023-08-livepeer/tree/main/contracts/bonding/BondingVotes.sol#L546



### <a href="#qa-summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Pausing withdrawals is unfair to the users

Users should always have the possibility of accessing their own funds, but when these functions are paused, their funds will be locked until the contracts are unpaused.

#### <ins>Proof Of Concept</ins>


```solidity
File: BondingManager.sol

249: function withdrawStake(uint256 _unbondingLockId) external whenSystemNotPaused currentRoundInitialized {
```

https://github.com/code-423n4/2023-08-livepeer/tree/main/contracts/bonding/BondingManager.sol#L249

```solidity
File: BondingManager.sol

273: function withdrawFees(address payable _recipient, uint256 _amount)
        external
        whenSystemNotPaused
        currentRoundInitialized
        autoClaimEarnings(msg.sender)
    {
```

https://github.com/code-423n4/2023-08-livepeer/tree/main/contracts/bonding/BondingManager.sol#L273



### <a href="#qa-summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Redundant inheritance specifier

The contracts below already extend the specified contract, so there is no need to list it in the inheritance list again

#### <ins>Proof Of Concept</ins>

```solidity
File: LivepeerGovernor.sol

26: contract LivepeerGovernor is
    ManagerProxyTarget,
    Initializable, // @audit
    GovernorUpgradeable,
    GovernorSettingsUpgradeable,
    GovernorTimelockControlUpgradeable,
    GovernorVotesUpgradeable,
    GovernorVotesQuorumFractionUpgradeable,
    GovernorCountingOverridable

```

https://github.com/code-423n4/2023-08-livepeer/tree/main/contracts/treasury/LivepeerGovernor.sol#L26



### <a href="#qa-summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Visibility should be set explicitly rather than defaulting to `internal`

#### <ins>Proof Of Concept</ins>


```solidity
File: BondingManager.sol

32: uint256 constant MAX_FUTURE_ROUND = 2**256 - 1;

```

https://github.com/code-423n4/2023-08-livepeer/tree/main/contracts/bonding/BondingManager.sol#L32