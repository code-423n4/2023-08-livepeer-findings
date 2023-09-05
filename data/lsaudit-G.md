# File: contracts/bonding/BondingVotes.sol

## Condition in `delegatorCumulativeStakeAt` can be optimized to save gas

* [File: contracts/bonding/BondingVotes.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingVotes.sol#L474-L478)

```
 if (rewardRound < bond.lastClaimRound) {
            // If the transcoder hasn't called reward() since the last time the delegator claimed earnings, there wil be
            // no rewards to add to the delegator's stake so we just return the originally bonded amount.
            return bond.bondedAmount;
        }
```

Function `delegatorCumulativeStakeAt` calculates cumulative stake of a delegator at any given round. Above condition checks if the transcoder hasn't called reward() since the last time the delegator claimed earnings. If he did (so `rewardRound < bond.lastClaimRound`), then there is no reward to add to the delegator's stake and no need to calculate it once again. We can just return the originally bonded amount
This condition, however, does not take into a consideration a scenario, when `rewardRound == bond.lastClaimRound`. In this case, the start and the end pools would be the same, so we don't need to calculate cumulative rewards once again. In that case, `bond.bondedAmount` should be returned also, when `rewardRound == bond.lastClaimRound`. 

To save some gas, we should include this scenario in the above if-condition. This line: `if (rewardRound < bond.lastClaimRound)` should be changed to: `if (rewardRound <= bond.lastClaimRound)`.

## Arithmetic which won't underflow/overflow can be `unchecked`

* [File: contracts/bonding/BondingVotes.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingVotes.sol#L98)

```
revert FutureLookup(_round, currentRound == 0 ? 0 : currentRound - 1);
```

When `currentRound` is 0, condition will return 0. Otherwise, `currentRound - 1` won't overflow, since `currentRound` is at least 1.



* [File: contracts/bonding/BondingVotes.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingVotes.sol)

```
156:        (uint256 amount, ) = getBondingStateAt(_account, clock() + 1);
182:        return getTotalActiveStakeAt(clock() + 1);
206:        (, address delegateAddress) = getBondingStateAt(_account, clock() + 1);
267:        if (_startRound != clock() + 1) {
268:            revert InvalidStartRound(_startRound, clock() + 1);
326:        if (_round > clock() + 1) {
327:            revert FutureLookup(_round, clock() + 1);
427:        if (_round > clock() + 1) {
428:            revert FutureLookup(_round, clock() + 1);

168:        (uint256 amount, ) = getBondingStateAt(_account, _round + 1);
195:        return getTotalActiveStakeAt(_round + 1);
219:        (, address delegateAddress) = getBondingStateAt(_account, _round + 1);
```

`clock() + 1` and `_round + 1` can be wrapped by `unchecked` block. It's almost impossible that `round` or `clock()` will ever overflow (this would require almost impossible to achieve amount of time to have `round` or `clock()` such big, that +1 will overflow it). Moreover, if those were so big and in a risk of overflow, the protocol would be broken - because achieving next round wouldn't be possible. Thus we can assume, that those values won't ever overflow and wrap them into `unchecked` block.


## Move functions which might revert before other operations
Assuming that function call which might revert does not interfere with other operations, move them as high possible


* [File: contracts/bonding/BondingVotes.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingVotes.sol#L287-L291)
```
checkpoints.data[_startRound] = bond;

// now store the startRound itself in the startRounds array to allow us
// to find it and lookup in the above mapping
checkpoints.startRounds.pushSorted(_startRound);
```
`pushSorted()` might revert when incorrect `_startRound` will be passed to it. To save some gas (in case `pushSorted()` will revert), you can move `checkpoints.startRounds.pushSorted(_startRound);` above `checkpoints.data[_startRound] = bond;`.

* [File: contracts/bonding/BondingVotes.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingVotes.sol#L308-L309)
```
        totalStakeCheckpoints.data[_round] = _totalStake;
        totalStakeCheckpoints.rounds.pushSorted(_round);
```

`pushSorted()` might revert when incorrect `_round` will be passed to it. To save some gas (in case `pushSorted()` will revert), you can call `totalStakeCheckpoints.rounds.pushSorted(_round);` before `totalStakeCheckpoints.data[_round] = _totalStake;`.

## Unnecessary variable declaration

* [File: contracts/bonding/BondingVotes.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingVotes.sol#L370-L374)
```
 bool isTranscoder = delegateAddress == _account;

        if (bond.bondedAmount == 0) {
            amount = 0;
        } else if (isTranscoder) {
```

Since `isTranscoder` is used only once, instead of wasting gas on `bool isTranscoder` declaration, `_account == delegate` can be used directly:

```
} else if (delegateAddress == _account) {
```

* [File: contracts/bonding/BondingVotes.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingVotes.sol#L398-L402)
```
bool isTranscoder = newDelegate == _account;
bool wasTranscoder = previousDelegate == _account;
// we want to register zero "delegate votes" when the account is/was not a transcoder
uint256 previousDelegateVotes = wasTranscoder ? previous.delegatedAmount : 0;
uint256 currentDelegateVotes = isTranscoder ? current.delegatedAmount : 0;
```
The same issue occurs with `isTranscored` and `wasTranscored` which might be used directly:

```
uint256 previousDelegateVotes = previousDelegate == _account ? previous.delegatedAmount : 0;
uint256 currentDelegateVotes = newDelegate == _account ? current.delegatedAmount : 0;
```



# File: contracts/bonding/BondingManager.sol

## Arithmetic which won't overflow/underflow can be wrapped by `unchecked` block

* [File: contracts/bonding/BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#281-L282)

```
        require(fees >= _amount, "insufficient fees to withdraw");
        delegators[msg.sender].fees = fees.sub(_amount);
```

The requirement statement guarantees that `fees >= _amount`, thus `fees - _amount` will never overflow. Instead of calling `fees.sub(_amount)`, `unchecked` block can be used. 

* [File: contracts/bonding/BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol)

```
511:                currentRound.add(1),
573:            del.startRound = currentRound.add(1);
585:            del.startRound = currentRound.add(1);
729:            newDel.startRound = currentRound.add(1);
827:        delegators[msg.sender].startRound = roundsManager().currentRound().add(1);
1270:        uint256 startRound = del.lastClaimRound.add(1);
1320:            uint256 nextRound = currRound.add(1);
1365:            uint256 nextRound = currRound.add(1);
1444:        uint256 deactivationRound = roundsManager().currentRound().add(1);
1506:        uint256 startRound = _lastClaimRound.add(1);
```

It's almost impossible that `currentRound` will ever overflow (this would require almost impossible to achieve amount of time to have `currentRound` such big value, that +1 will overflow it). Moreover, if those values were so big and in a risk of overflow, the protocol would be broken - because achieving next round wouldn't be possible. Thus we can assume, that those values won't ever overflow and instead of using `.add(1)`, we can use ` + 1` wrapped in `unchecked` block.

## Move checks which might revert at the beginning of the function

* [File: contracts/bonding/BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L250-L257)

```
  Delegator storage del = delegators[msg.sender];
        UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];

        require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID");
        require(
            lock.withdrawRound <= roundsManager().currentRound(),
            "withdraw round must be before or equal to the current round"
        );
```

require conditions which might revert earlier can be moved on top:

```
require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID");
UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];
 require(
         lock.withdrawRound <= roundsManager().currentRound(),
         "withdraw round must be before or equal to the current round"
        );
Delegator storage del = delegators[msg.sender];
```

* [File: contracts/bonding/BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L848-L850) 
```
uint256 currentRound = roundsManager().currentRound();

require(isActiveTranscoder(msg.sender), "caller must be an active transcoder");
```

require condition which might revert can be moved on top:

```
require(isActiveTranscoder(msg.sender), "caller must be an active transcoder");

uint256 currentRound = roundsManager().currentRound();
```

* [File: contracts/bonding/BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L1570-L1573) 
```
Delegator storage del = delegators[_delegator];
UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];

require(isValidUnbondingLock(_delegator, _unbondingLockId), "invalid unbonding lock ID");
```

require condition which might revert can be moved on top:

```
require(isValidUnbondingLock(_delegator, _unbondingLockId), "invalid unbonding lock ID");
Delegator storage del = delegators[_delegator];
UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];

```


# File: contracts/treasury/GovernorCountingOverridable.sol

## Unnecessary variable declaration

* [File: contracts/treasury/GovernorCountingOverridable.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L110-L112)
```
uint256 totalVotes = againstVotes + forVotes + abstainVotes;

return totalVotes >= quorum(proposalSnapshot(_proposalId));
```

Since `totalVotes` is used only once, instead of wasting gas on `uint256 totalVotes` declaration, `againstVotes + forVotes + abstainVotes` can be used directly:

```
return againstVotes + forVotes + abstainVotes >= quorum(proposalSnapshot(_proposalId));
```


* [File: contracts/treasury/GovernorCountingOverridable.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L122-L124)
```
uint256 opinionatedVotes = againstVotes + forVotes;

return forVotes >= MathUtils.percOf(opinionatedVotes, quota);
```

Since `opinionatedVotes` is used only once, instead of wasting gas on `uint256 opinionatedVotes` declaration, `againstVotes + forVotes` can be used directly:

```
return forVotes >= MathUtils.percOf(againstVotes + forVotes, quota);
```

* [File: contracts/treasury/GovernorCountingOverridable.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L184-185)
```
        bool isTranscoder = _account == delegate;
        if (isTranscoder) {
```

Since `isTranscoder` is used only once, instead of wasting gas on `bool isTranscoder` declaration, `_account == delegate` can be used directly:

```
if (_account == delegate) {
```

## Move checks which might revert at the beginning of the function

* [File: contracts/treasury/GovernorCountingOverridable.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L137-L147)

```
if (_supportInt > uint8(VoteType.Abstain)) {
    revert InvalidVoteType(_supportInt);
}
VoteType support = VoteType(_supportInt);

ProposalTally storage tally = _proposalTallies[_proposalId];
ProposalVoterState storage voter = tally.voters[_account];

if (voter.hasVoted) {
    revert VoteAlreadyCast();
}
```

`VoteType support = VoteType(_supportInt);` line can be moved after the second revert:

```
if (_supportInt > uint8(VoteType.Abstain)) {
    revert InvalidVoteType(_supportInt);
}

ProposalTally storage tally = _proposalTallies[_proposalId];
ProposalVoterState storage voter = tally.voters[_account];

if (voter.hasVoted) {
    revert VoteAlreadyCast();
}

VoteType support = VoteType(_supportInt);
```

## Unnecessary assertion in `_handleVoteOverrides`


* [File: contracts/treasury/GovernorCountingOverridable.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L201-208)

```
if (delegateSupport == VoteType.Against) {
    _tally.againstVotes -= _weight;
} else if (delegateSupport == VoteType.For) {
    _tally.forVotes -= _weight;
} else {
    assert(delegateSupport == VoteType.Abstain);
    _tally.abstainVotes -= _weight;
}
```

`assert(delegateSupport == VoteType.Abstain)` is not needed, because if `delegateSupport` is not `VoteType.Against` (first if-condition), is not `VoteType.For` (second else-if condition), then it must be `VoteType.Abstain`.
Function `_handleVoteOverrides` is called by `_countVote` and inside `_countVote` there is already a check that: 

```
if (_supportInt > uint8(VoteType.Abstain)) {
    revert InvalidVoteType(_supportInt);
}
```

This implies, that when we reach else-condition, then `delegateSupport` must be `VoteType.Abstain`. The `assert` is not needed.


# File: contracts/bonding/libraries/SortedArrays.sol

## `array.length` is calculated twice

* [File: contracts/bonding/libraries/SortedArrays.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/libraries/SortedArrays.sol#L65-L68)
```
        if (array.length == 0) {
            array.push(val);
        } else {
            uint256 last = array[array.length - 1];
```

Cache the value of `array.length` instead of calculating it twice.

# File: contracts/bonding/libraries/EarningsPoolLIP36.sol

## `PreciseMathUtils.percPoints(1, 1)` can be pre-calculated before contract deployment

Since `PreciseMathUtils.percPoints(1, 1)` contains constant value, its value can be calculated before compiling and deploying the contract.

* [File: contracts/bonding/libraries/EarningsPoolLIP36.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/libraries/EarningsPoolLIP36.sol#L26)
```
: PreciseMathUtils.percPoints(1, 1);
```

* [File: contracts/bonding/libraries/EarningsPoolLIP36.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/libraries/EarningsPoolLIP36.sol#L54)
```
            : PreciseMathUtils.percPoints(1, 1);
```

* [File: contracts/bonding/libraries/EarningsPoolLIP36.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/libraries/EarningsPoolLIP36.sol#L79)
```
 _startPool.cumulativeRewardFactor = PreciseMathUtils.percPoints(1, 1);
```

* [File: contracts/bonding/libraries/EarningsPoolLIP36.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/libraries/EarningsPoolLIP36.sol#L84)
```
 _endPool.cumulativeRewardFactor = PreciseMathUtils.percPoints(1, 1);
```

