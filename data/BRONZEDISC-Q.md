
## QA
---


### Function Visibility [1]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol

```solidity
// place these internal functions for last since there are no private ones.
101:    function bondingVotes() internal view returns (IVotes) {
108:    function treasury() internal view returns (Treasury) {
132:    function _execute(
142:    function _cancel(
151:    function _executor()
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol

```solidity
// place these internal functions for last since there are no private ones.
64:    function __GovernorCountingOverridable_init(uint256 _quota) internal onlyInitializing {
68:    function __GovernorCountingOverridable_init_unchained(uint256 _quota) internal onlyInitializing {

// place this public function with the other ones
217:    function votes() public view virtual returns (IVotes);
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol

```solidity
// place this internal view after the non-view internal
28:    function findLowerBound(uint256[] storage _array, uint256 _val) internal view returns (uint256) {
```

---

### natSpec missing [2]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol

```solidity
// @params/return missing
54:    function initialize(
78:    function quorumDenominator() public view virtual override returns (uint256) {
85:    function votes() public view override returns (IVotes) {
101:    function bondingVotes() internal view returns (IVotes) {
114:    function proposalThreshold()
123:    function state(uint256 proposalId)
132:    function _execute(
142:    function _cancel(
151:    function _executor()
160:    function supportsInterface(bytes4 interfaceId)
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/Treasury.sol

```solidity
16:    function initialize(
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol

```solidity
22:    error InvalidVoteType(uint8 voteType);
23:    error VoteAlreadyCast();
64:    function __GovernorCountingOverridable_init(uint256 _quota) internal onlyInitializing {
68:    function __GovernorCountingOverridable_init_unchained(uint256 _quota) internal onlyInitializing {

// @return/@param missing
37:    struct ProposalVoterState {
48:    struct ProposalTally {
76:    function COUNTING_MODE() public pure virtual override returns (string memory) {
83:    function hasVoted(uint256 _proposalId, address _account) public view virtual override returns (bool) {
90:    function proposalVotes(uint256 _proposalId)
107:    function _quorumReached(uint256 _proposalId) internal view virtual override returns (bool) {
118:    function _voteSucceeded(uint256 _proposalId) internal view virtual override returns (bool) {
130:    function _countVote(
217:    function votes() public view virtual returns (IVotes);

// @return missing
174:    function _handleVoteOverrides(
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol

```solidity
15:    error DecreasingValues(uint256 newValue, uint256 lastValue);
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol

```solidity
9:  library EarningsPoolLIP36 {
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol

```solidity
// place this public function right after all the external ones.
137:    function clock() public view returns (uint48) {
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol

```solidity
135:    modifier autoCheckpoint(address _account) {

// @params missing
273:    function withdrawFees(address payable _recipient, uint256 _amount)
1176:    function _setTreasuryRewardCutRate(uint256 _cutRate) internal {
1294:    function increaseTotalStake(
1307:    function increaseTotalStakeUncheckpointed(
1352:    function decreaseTotalStake(
1392:    function tryToJoinActiveSet(
1437:    function resignTranscoder(address _transcoder) internal {
1591:    function _checkpointBondingState(
1643:    function treasury() internal view returns (address) {
1647:    function bondingVotes() internal view returns (IBondingVotes) {
1651:    function _onlyTicketBroker() internal view {
1655:    function _onlyRoundsManager() internal view {
1659:    function _onlyVerifier() internal view {
1663:    function _currentRoundInitialized() internal view {
1667:    function _autoClaimEarnings(address _delegator) internal {

// @return missing
1189:    function cumulativeFactorsPool(Transcoder storage _transcoder, uint256 _round)
```


---

### State variable and function names [3]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol

```solidity
//  private and internal `functions` should preppend with `underline`
101:    function bondingVotes() internal view returns (IVotes) {
108:    function treasury() internal view returns (Treasury) {
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol

```solidity
//  private and internal `functions` should preppend with `underline`
28:    function findLowerBound(uint256[] storage _array, uint256 _val) internal view returns (uint256) {
64:    function pushSorted(uint256[] storage array, uint256 val) internal {
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol

```solidity
// private and internal `functions` should preppend with `underline`
18:    function updateCumulativeFeeFactor(
47:    function updateCumulativeRewardFactor(
71:    function delegatorCumulativeStakeAndFees(
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol

```solidity
// private and internal `variables` should preppend with `underline`
78:    mapping(address => BondingCheckpointsByRound) private bondingCheckpoints;
82:    TotalActiveStakeByRound private totalStakeCheckpoints;

//  private and internal `functions` should preppend with `underline`
387:    function onBondingCheckpointChanged(
422:    function getBondingCheckpointAt(address _account, uint256 _round)
459:    function delegatorCumulativeStakeAt(BondingCheckpoint storage bond, uint256 _round)
499:    function getLastTranscoderRewardsEarningsPool(address _transcoder, uint256 _round)
520:    function getTranscoderEarningsPoolForRound(address _transcoder, uint256 _round)
539:    function bondingManager() internal view returns (IBondingManager) {
546:    function roundsManager() internal view returns (IRoundsManager) {
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol

```solidity
//  private and internal `variables` should preppend with `underline`
84:    mapping(address => Delegator) private delegators;
85:    mapping(address => Transcoder) private transcoders;
95:    SortedDoublyLL.Data private transcoderPool;

//  private and internal `functions` should preppend with `underline`
1238:    function delegatorCumulativeStakeAndFees(
1294:    function increaseTotalStake(
1307:    function increaseTotalStakeUncheckpointed(
1352:    function decreaseTotalStake(
1392:    function tryToJoinActiveSet(
1437:    function resignTranscoder(address _transcoder) internal {
1459:    function updateTranscoderWithRewards(
1500:    function updateDelegatorWithEarnings(
1564:    function processRebond(
1615:    function livepeerToken() internal view returns (ILivepeerToken) {
1623:    function minter() internal view returns (IMinter) {
1631:    function l2Migrator() internal view returns (address) {
1639:    function roundsManager() internal view returns (IRoundsManager) {
1643:    function treasury() internal view returns (address) {
1647:    function bondingVotes() internal view returns (IBondingVotes) {
```


