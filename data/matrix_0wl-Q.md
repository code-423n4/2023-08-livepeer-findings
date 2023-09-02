## Non Critical Issues

|       | Issue                                                         |
| ----- | :------------------------------------------------------------ |
| NC-1  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                          |
| NC-2  | GENERATE PERFECT CODE HEADERS EVERY TIME                      |
| NC-3  | INCONSISTENT SOLIDITY VERSIONS                                |
| NC-4  | IMPLEMENTATION CONTRACT MAY NOT BE INITIALIZED                |
| NC-5  | MARK VISIBILITY OF INITIALIZE(…) FUNCTIONS AS EXTERNAL        |
| NC-6  | LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS  |
| NC-7  | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION |
| NC-8  | Missing checks for `address(0)`                               |
| NC-9  | MISSING EVENT FOR CRITICAL PARAMETER CHANGE                   |
| NC-10 | NO SAME VALUE INPUT CONTROL                                   |
| NC-11 | Omissions in events                                           |
| NC-12 | SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC            |
| NC-13 | USE A MORE RECENT VERSION OF SOLIDITY                         |

### [NC-1] ADD TO BLACKLIST FUNCTION

#### Description:

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract
https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

#### Recommended Mitigation Steps:

Add to Blacklist function and modifier.

### [NC-2] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-3] INCONSISTENT SOLIDITY VERSIONS

#### Description:

The project is compiled with different versions of Solidity, which is not recommended because it can lead to undefined behaviors.

It is better to use one Solidity compiler version across all contracts instead of different versions with different bugs and security checks.

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/BondingVotes.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/IBondingManager.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/IBondingVotes.sol

2: pragma solidity ^0.8.9;

```

```solidity
File: contracts/bonding/libraries/EarningsPoolLIP36.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/libraries/SortedArrays.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/IVotes.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/LivepeerGovernor.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/Treasury.sol

2: pragma solidity 0.8.9;

```

### [NC-4] IMPLEMENTATION CONTRACT MAY NOT BE INITIALIZED

#### Description:

OpenZeppelin recommends that the initializer modifier be applied to constructors.

Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.

https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

#### **Proof Of Concept**

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

4: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

21: abstract contract GovernorCountingOverridable is Initializable, GovernorUpgradeable {

```

```solidity
File: contracts/treasury/LivepeerGovernor.sol

4: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

28:     Initializable,

```

```solidity
File: contracts/treasury/Treasury.sol

15: contract Treasury is Initializable, TimelockControllerUpgradeable {

```

### [NC-5] MARK VISIBILITY OF INITIALIZE(…) FUNCTIONS AS EXTERNAL

#### Description:

If someone wants to extend via inheritance, it might make more sense that the overridden initialize(...) function calls the internal {...}\_init function, not the parent public initialize(...) function.

External instead of public would give more sense of the initialize(...) functions to behave like a constructor (only called on deployment, so should only be called externally).

From a security point of view, it might be safer so that it cannot be called internally by accident in the child contract.

#### **Proof Of Concept**

```solidity
File: contracts/treasury/LivepeerGovernor.sol

54:     function initialize(

```

```solidity
File: contracts/treasury/Treasury.sol

16:     function initialize(

```

### [NC-6] LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS

#### Description:

To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the initialize() functions:

#### **Proof Of Concept**

```solidity
File: contracts/treasury/LivepeerGovernor.sol

54:     function initialize(

```

```solidity
File: contracts/treasury/Treasury.sol

16:     function initialize(

```

### [NC-7] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/BondingVotes.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/IBondingManager.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/IBondingVotes.sol

2: pragma solidity ^0.8.9;

```

```solidity
File: contracts/bonding/libraries/EarningsPoolLIP36.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/libraries/SortedArrays.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/IVotes.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/LivepeerGovernor.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/Treasury.sol

2: pragma solidity 0.8.9;

```

#### Recommended Mitigation Steps:

Use solidity pragma version min. 0.8.13

### [NC-8] Missing checks for `address(0)`

#### Description:

0 address control should be done in these parts;

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

62:         address delegateAddress; // The address delegated to

62:         address delegateAddress; // The address delegated to

84:     mapping(address => Delegator) private delegators;

85:     mapping(address => Transcoder) private transcoders;

130:     modifier autoClaimEarnings(address _delegator) {

135:     modifier autoCheckpoint(address _account) {

149:     constructor(address _controller) Manager(_controller) {}

156:         unbondingPeriod = _unbondingPeriod;

177:         treasuryBalanceCeiling = _ceiling;

199:         transcoderWithHint(_rewardCut, _feeShare, address(0), address(0));

199:         transcoderWithHint(_rewardCut, _feeShare, address(0), address(0));

207:     function bond(uint256 _amount, address _to) external {

208:         bondWithHint(_amount, _to, address(0), address(0), address(0), address(0));

208:         bondWithHint(_amount, _to, address(0), address(0), address(0), address(0));

208:         bondWithHint(_amount, _to, address(0), address(0), address(0), address(0));

208:         bondWithHint(_amount, _to, address(0), address(0), address(0), address(0));

216:         unbondWithHint(_amount, address(0), address(0));

216:         unbondWithHint(_amount, address(0), address(0));

224:         rebondWithHint(_unbondingLockId, address(0), address(0));

224:         rebondWithHint(_unbondingLockId, address(0), address(0));

232:     function rebondFromUnbonded(address _to, uint256 _unbondingLockId) external {

233:         rebondFromUnbondedWithHint(_to, _unbondingLockId, address(0), address(0));

233:         rebondFromUnbondedWithHint(_to, _unbondingLockId, address(0), address(0));

241:     function checkpointBondingState(address _account) external {

273:     function withdrawFees(address payable _recipient, uint256 _amount)

279:         require(_recipient != address(0), "invalid recipient");

281:         require(fees >= _amount, "insufficient fees to withdraw");

294:         rewardWithHint(address(0), address(0));

294:         rewardWithHint(address(0), address(0));

303:         address _transcoder,

369:         uint256 transcoderCommissionFees = _fees.sub(delegatorsFees);

395:         address _transcoder,

396:         address _finder,

424:             if (_finder != address(0)) {

436:                 emit TranscoderSlashed(_transcoder, address(0), penalty, 0);

488:         address _newPosPrev,

489:         address _newPosNext

504:         t.rewardCut = _rewardCut;

505:         t.feeShare = _feeShare;

539:         address _owner,

540:         address _to,

541:         address _oldDelegateNewPosPrev,

542:         address _oldDelegateNewPosNext,

543:         address _currDelegateNewPosPrev,

544:         address _currDelegateNewPosNext

552:         uint256 delegationAmount = _amount;

554:         address currentDelegate = del.delegateAddress;

559:         if (msg.sender != _owner && msg.sender != l2Migrator()) {

563:                 require(_to != _owner, "INVALID_DELEGATE");

565:                 require(currentDelegate == _to, "INVALID_DELEGATE_CHANGE");

576:         } else if (currentBondedAmount > 0 && currentDelegate != _to) {

582:             require(!isRegisteredTranscoder(_owner), "registered transcoders can't delegate towards other addresses");

608:         del.delegateAddress = _to;

616:             livepeerToken().transferFrom(msg.sender, address(minter()), _amount);

642:         address _to,

643:         address _oldDelegateNewPosPrev,

644:         address _oldDelegateNewPosNext,

645:         address _currDelegateNewPosPrev,

646:         address _currDelegateNewPosNext

680:         address _delegator,

682:         address _oldDelegateNewPosPrev,

683:         address _oldDelegateNewPosNext,

684:         address _newDelegateNewPosPrev,

685:         address _newDelegateNewPosNext

693:         address oldDelDelegate = oldDel.delegateAddress;

719:         if (newDel.delegateAddress == address(0) && newDel.bondedAmount == 0) {

722:             require(oldDelDelegate != _delegator, "INVALID_DELEGATOR");

747:         address _newPosPrev,

748:         address _newPosNext

757:         address currentDelegate = del.delegateAddress;

771:             del.delegateAddress = address(0);

798:         address _newPosPrev,

799:         address _newPosNext

819:         address _to,

821:         address _newPosPrev,

822:         address _newPosNext

829:         delegators[msg.sender].delegateAddress = _to;

842:     function rewardWithHint(address _newPosPrev, address _newPosNext)

842:     function rewardWithHint(address _newPosPrev, address _newPosNext)

885:             address trsry = treasury();

908:     function pendingStake(address _delegator, uint256 _endRound) public view returns (uint256) {

923:     function pendingFees(address _delegator, uint256 _endRound) public view returns (uint256) {

937:     function transcoderTotalStake(address _transcoder) public view returns (uint256) {

946:     function transcoderStatus(address _transcoder) public view returns (TranscoderStatus) {

956:     function delegatorStatus(address _delegator) public view returns (DelegatorStatus) {

987:     function getTranscoder(address _transcoder)

1027:     function getTranscoderEarningsPoolForRound(address _transcoder, uint256 _round)

1058:     function getDelegator(address _delegator)

1064:             address delegateAddress,

1089:     function getDelegatorUnbondingLock(address _delegator, uint256 _unbondingLockId)

1119:     function getFirstTranscoderInPool() public view returns (address) {

1128:     function getNextTranscoderInPool(address _transcoder) public view returns (address) {

1128:     function getNextTranscoderInPool(address _transcoder) public view returns (address) {

1145:     function isActiveTranscoder(address _transcoder) public view returns (bool) {

1156:     function isRegisteredTranscoder(address _transcoder) public view returns (bool) {

1158:         return d.delegateAddress == _transcoder && d.bondedAmount > 0;

1167:     function isValidUnbondingLock(address _delegator, uint256 _unbondingLockId) public view returns (bool) {

1179:         nextRoundTreasuryRewardCutRate = _cutRate;

1194:         pool.cumulativeRewardFactor = _transcoder.earningsPoolPerRound[_round].cumulativeRewardFactor;

1195:         pool.cumulativeFeeFactor = _transcoder.earningsPoolPerRound[_round].cumulativeFeeFactor;

1213:         uint256 lastRewardRound = _transcoder.lastRewardRound;

1219:         uint256 lastFeeRound = _transcoder.lastFeeRound;

1259:     function pendingStakeAndFees(address _delegator, uint256 _endRound)

1271:         address delegateAddr = del.delegateAddress;

1272:         bool isTranscoder = _delegator == delegateAddr;

1275:         if (startRound <= _endRound) {

1295:         address _delegate,

1297:         address _newPosPrev,

1298:         address _newPosNext

1308:         address _delegate,

1310:         address _newPosPrev,

1311:         address _newPosNext

1353:         address _delegate,

1355:         address _newPosPrev,

1356:         address _newPosNext

1393:         address _transcoder,

1396:         address _newPosPrev,

1397:         address _newPosNext

1402:             address lastTranscoder = transcoderPool.getLast();

1417:             transcoders[lastTranscoder].deactivationRound = _activationRound;

1426:         t.lastActiveStakeUpdateRound = _activationRound;

1427:         t.activationRound = _activationRound;

1437:     function resignTranscoder(address _transcoder) internal {

1460:         address _transcoder,

1463:         address _newPosPrev,

1464:         address _newPosNext

1473:         uint256 delegatorsRewards = _rewards.sub(transcoderCommissionRewards);

1501:         address _delegator,

1506:         uint256 startRound = _lastClaimRound.add(1);

1512:         if (del.delegateAddress != address(0)) {

1534:             if (del.delegateAddress == _delegator) {

1550:         del.lastClaimRound = _endRound;

1565:         address _delegator,

1567:         address _newPosPrev,

1568:         address _newPosNext

1592:         address _owner,

1631:     function l2Migrator() internal view returns (address) {

1643:     function treasury() internal view returns (address) {

1667:     function _autoClaimEarnings(address _delegator) internal {

```

```solidity
File: contracts/bonding/BondingVotes.sol

32:         address delegateAddress;

78:     mapping(address => BondingCheckpointsByRound) private bondingCheckpoints;

107:     constructor(address _controller) Manager(_controller) {}

155:     function getVotes(address _account) external view returns (uint256) {

167:     function getPastVotes(address _account, uint256 _round) external view onlyPastRounds(_round) returns (uint256) {

205:     function delegates(address _account) external view returns (address) {

205:     function delegates(address _account) external view returns (address) {

206:         (, address delegateAddress) = getBondingStateAt(_account, clock() + 1);

218:     function delegatedAt(address _account, uint256 _round) external view onlyPastRounds(_round) returns (address) {

218:     function delegatedAt(address _account, uint256 _round) external view onlyPastRounds(_round) returns (address) {

219:         (, address delegateAddress) = getBondingStateAt(_account, _round + 1);

226:     function delegate(address) external pure {

234:         address,

259:         address _account,

262:         address _delegateAddress,

269:         } else if (_lastClaimRound >= _startRound) {

308:         totalStakeCheckpoints.data[_round] = _totalStake;

315:     function hasCheckpoint(address _account) public view returns (bool) {

361:     function getBondingStateAt(address _account, uint256 _round)

365:         returns (uint256 amount, address delegateAddress)

370:         bool isTranscoder = delegateAddress == _account;

388:         address _account,

392:         address previousDelegate = previous.delegateAddress;

393:         address newDelegate = current.delegateAddress;

398:         bool isTranscoder = newDelegate == _account;

399:         bool wasTranscoder = previousDelegate == _account;

422:     function getBondingCheckpointAt(address _account, uint256 _round)

499:     function getLastTranscoderRewardsEarningsPool(address _transcoder, uint256 _round)

520:     function getTranscoderEarningsPoolForRound(address _transcoder, uint256 _round)

554:         if (msg.sender != address(bondingManager())) {

555:             revert InvalidCaller(msg.sender, address(bondingManager()));

```

```solidity
File: contracts/bonding/IBondingManager.sol

9:     event TranscoderUpdate(address indexed transcoder, uint256 rewardCut, uint256 feeShare);

10:     event TranscoderActivated(address indexed transcoder, uint256 activationRound);

11:     event TranscoderDeactivated(address indexed transcoder, uint256 deactivationRound);

12:     event TranscoderSlashed(address indexed transcoder, address finder, uint256 penalty, uint256 finderReward);

12:     event TranscoderSlashed(address indexed transcoder, address finder, uint256 penalty, uint256 finderReward);

13:     event Reward(address indexed transcoder, uint256 amount);

14:     event TreasuryReward(address indexed transcoder, address treasury, uint256 amount);

14:     event TreasuryReward(address indexed transcoder, address treasury, uint256 amount);

16:         address indexed newDelegate,

17:         address indexed oldDelegate,

18:         address indexed delegator,

23:         address indexed delegate,

24:         address indexed delegator,

29:     event Rebond(address indexed delegate, address indexed delegator, uint256 unbondingLockId, uint256 amount);

29:     event Rebond(address indexed delegate, address indexed delegator, uint256 unbondingLockId, uint256 amount);

31:         address indexed oldDelegator,

32:         address indexed newDelegator,

37:     event WithdrawStake(address indexed delegator, uint256 unbondingLockId, uint256 amount, uint256 withdrawRound);

38:     event WithdrawFees(address indexed delegator, address recipient, uint256 amount);

38:     event WithdrawFees(address indexed delegator, address recipient, uint256 amount);

40:         address indexed delegate,

41:         address indexed delegator,

60:         address _transcoder,

66:         address _transcoder,

67:         address _finder,

77:     function transcoderTotalStake(address _transcoder) external view returns (uint256);

79:     function isActiveTranscoder(address _transcoder) external view returns (bool);

85:     function getTranscoderEarningsPoolForRound(address _transcoder, uint256 _round)

```

```solidity
File: contracts/bonding/IBondingVotes.sol

10:     error InvalidCaller(address caller, address required);

10:     error InvalidCaller(address caller, address required);

16:     error MissingEarningsPool(address transcoder, uint256 round);

27:     event DelegatorBondedAmountChanged(address indexed delegate, uint256 previousBondedAmount, uint256 newBondedAmount);

32:         address _account,

35:         address _delegateAddress,

45:     function hasCheckpoint(address _account) external view returns (bool);

49:     function getBondingStateAt(address _account, uint256 _round)

52:         returns (uint256 amount, address delegateAddress);

```

```solidity
File: contracts/bonding/libraries/EarningsPoolLIP36.sol

23:         uint256 prevCumulativeFeeFactor = _prevEarningsPool.cumulativeFeeFactor;

24:         uint256 prevCumulativeRewardFactor = _prevEarningsPool.cumulativeRewardFactor != 0

52:         uint256 prevCumulativeRewardFactor = _prevEarningsPool.cumulativeRewardFactor != 0

87:         cFees = _fees.add(

```

```solidity
File: contracts/bonding/libraries/SortedArrays.sol

29:         uint256 len = _array.length;

34:         if (_array[len - 1] <= _val) {

38:         uint256 upperIdx = _array.findUpperBound(_val);

44:         if (_array[upperIdx] == _val) {

```

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

6: import "@openzeppelin/contracts-upgradeable/interfaces/IERC5805Upgradeable.sol";

52:         mapping(address => ProposalVoterState) voters;

69:         quota = _quota;

83:     function hasVoted(uint256 _proposalId, address _account) public view virtual override returns (bool) {

100:         ProposalTally storage tally = _proposalTallies[_proposalId];

132:         address _account,

142:         ProposalTally storage tally = _proposalTallies[_proposalId];

151:         _weight = _handleVoteOverrides(_proposalId, tally, voter, _account, _weight);

154:             tally.againstVotes += _weight;

156:             tally.forVotes += _weight;

158:             tally.abstainVotes += _weight;

178:         address _account,

182:         address delegate = votes().delegatedAt(_account, timepoint);

184:         bool isTranscoder = _account == delegate;

192:         ProposalVoterState storage delegateVoter = _tally.voters[delegate];

193:         delegateVoter.deductions += _weight;

202:                 _tally.againstVotes -= _weight;

204:                 _tally.forVotes -= _weight;

207:                 _tally.abstainVotes -= _weight;

```

```solidity
File: contracts/treasury/IVotes.sol

4: import "@openzeppelin/contracts-upgradeable/interfaces/IERC5805Upgradeable.sol";

6: interface IVotes is IERC5805Upgradeable {

9:     function delegatedAt(address account, uint256 timepoint) external returns (address);

9:     function delegatedAt(address account, uint256 timepoint) external returns (address);

```

```solidity
File: contracts/treasury/LivepeerGovernor.sol

43:     constructor(address _controller) Manager(_controller) {

79:         return MathUtils.PERC_DIVISOR;

134:         address[] memory targets,

143:         address[] memory targets,

155:         returns (address)

```

```solidity
File: contracts/treasury/Treasury.sol

18:         address[] memory proposers,

19:         address[] memory executors,

20:         address admin

```

#### Recommended Mitigation Steps:

Add code like this: `if (oracle == address(0)) revert ADDRESS_ZERO();` or `require(address(\_VARIABLE) != address(0), "Address cannot be zero");

### [NC-9] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

#### Description:

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

158:         emit ParameterUpdate("unbondingPeriod");

179:         emit ParameterUpdate("treasuryBalanceCeiling");


189:         emit ParameterUpdate("numActiveTranscoders");

468:             emit ParameterUpdate("treasuryRewardCutRate");

```

### [NC-10] NO SAME VALUE INPUT CONTROL

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

156:         unbondingPeriod = _unbondingPeriod;

```

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

69:         quota = _quota;

```

### [NC-11] Omissions in events

#### Description:

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

158:         emit ParameterUpdate("unbondingPeriod");

179:         emit ParameterUpdate("treasuryBalanceCeiling");

189:         emit ParameterUpdate("numActiveTranscoders");

1181:         emit ParameterUpdate("nextRoundTreasuryRewardCutRate");


```

### [NC-12] SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC

#### Description:

Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

#### Exploit Scenario:

A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

#### Recommendation:

Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug. Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

#### **Proof Of Concept**

```solidity
File: hardhat.config.ts

solidity: {
        compilers: [
            {
                version: "0.8.9",
                settings: {
                    optimizer: {
                        enabled: true,
                        runs: 200
                    }
                }
            }
        ]
    },
```

### [NC-13] USE A MORE RECENT VERSION OF SOLIDITY

#### Description:

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

https://swcregistry.io/docs/SWC-102

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/BondingVotes.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/IBondingManager.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/IBondingVotes.sol

2: pragma solidity ^0.8.9;

```

```solidity
File: contracts/bonding/libraries/EarningsPoolLIP36.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/libraries/SortedArrays.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/IVotes.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/LivepeerGovernor.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/Treasury.sol

2: pragma solidity 0.8.9;

```

## Low Issues

|     | Issue                                                             |
| --- | :---------------------------------------------------------------- |
| L-1 | Initializing state-variables in proxy-based upgradeable contracts |
| L-2 | The critical parameters in initialize(...) are not set safely     |
| L-3 | INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY                    |
| L-4 | UNSAFE CAST                                                       |
| L-5 | Unsafe ERC20 operation(s)                                         |

### [L-1] Initializing state-variables in proxy-based upgradeable contracts

#### Description:

This should be done in initializer functions and not as part of the state variable declarations in which case they won’t be set.

https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#avoid-initial-values-in-field-declarations

#### **Proof Of Concept**

```solidity
File: contracts/treasury/LivepeerGovernor.sol

54:     function initialize(

```

```solidity
File: contracts/treasury/Treasury.sol

16:     function initialize(

```

### [L-2] The critical parameters in initialize(...) are not set safely

#### **Proof Of Concept**

```solidity
File: contracts/treasury/LivepeerGovernor.sol

54:     function initialize(

```

```solidity
File: contracts/treasury/Treasury.sol

16:     function initialize(

```

### [L-3] INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY

#### Description:

`initialize()` function can be called anybody when the contract is not initialized.

#### **Proof Of Concept**

```solidity
File: contracts/treasury/LivepeerGovernor.sol

60:     ) public initializer {

```

```solidity
File: contracts/treasury/Treasury.sol

21:     ) external initializer {

```

### [L-4] UNSAFE CAST

#### Description:

For example uint64(n) where n is uint256 OR uint32(block.timestamp)

Keep in mind that the version of solidity used, despite being greater than 0.8, does not prevent integer overflows during casting, it only does so in mathematical operations.

It is necessary to safely convert between the different numeric types.

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

Use a safeCast from Open Zeppelin or increase the type length.

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

137:         if (_supportInt > uint8(VoteType.Abstain)) {

```

### [L-5] Unsafe ERC20 operation(s)

#### Description:

There are ERC20 tokens that charge fee for every transfer() / transferFrom().

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

616:             livepeerToken().transferFrom(msg.sender, address(minter()), _amount);

```
