## [G-01] High Gas Costs in checkpointTotalActiveStake
**Location:** Inside the checkpointTotalActiveStake function.

**Description:** The function may consume significant gas when there are many total active stake checkpoints, especially if the round provided is not recent.

**Code snippet:**


uint256[] storage initializedRounds = totalStakeCheckpoints.rounds;
uint256 upper = initializedRounds.findUpperBound(_round);
if (upper == 0) {
    return 0;
} else if (upper < initializedRounds.length) {
    uint256 nextInitedRound = initializedRounds[upper];
    return totalStakeCheckpoints.data[nextInitedRound];
} else {
    return bondingManager().nextRoundTotalActiveStake();
}

**Recommended Change:**


uint256 currentRound = clock();
if (_round == currentRound) {
    return totalStakeCheckpoints.data[_round];
} else if (_round < currentRound && totalStakeCheckpoints.data[_round] > 0) {
    return totalStakeCheckpoints.data[_round];
} else if (_round > currentRound) {
    return bondingManager().nextRoundTotalActiveStake();
}

## [G-02] Redundant Gas Check in getBondingStateAt
**Location:** Inside the getBondingStateAt function.

**Description:** The function checks _round against the current round twice, which is redundant.

 **Code snippet:**


if (_round > clock() + 1) {
    revert FutureLookup(_round, clock() + 1);
}

**Recommended Change:**


uint256 currentRound = clock();
if (_round > currentRound + 1) {
    revert FutureLookup(_round, currentRound + 1);
}


## [G-03] Redundant Storage Reads in delegatorCumulativeStakeAt
**Location:** Inside the delegatorCumulativeStakeAt function.

**Description:** The function reads from storage multiple times, which can be optimized to reduce gas consumption.

 **Code snippet:**


BondingCheckpoint storage bond = getBondingCheckpointAt(_account, _round);

**Recommended Change:**


BondingCheckpoint storage bond = getBodingCheckpointAt(_account, _round);
if (bond.bondedAmount == 0) {
    amount = 0;
} else if (isTranscoder) {
    amount = bond.delegatedAmount;
} else {
    amount = delegatorCumulativeStakeAt(bond, _round);
}

## [G-04] Redundant Delegate Address Check in getBondingStateAt
**Location:** Inside the getBondingStateAt function.

**Description:** The function checks if _account is a transcoder twice, which is redundant.

**Code snippet:**


bool isTranscoder = delegateAddress == _account;
if (isTranscoder) {
    // ...
}

**Recommended Change:**


bool isTranscoder = delegateAddress == _account;
if (isTranscoder) {
    // ...
} else {
    // ...
}

## [G-05] Redundant Event Emission in onBondingCheckpointChanged
**Location:** Inside the onBondingCheckpointChanged function.

**Description:** The function emits events unconditionally, which can lead to redundant event emissions.

**Code snippet:**


emit DelegateChanged(_account, previousDelegate, newDelegate);
emit DelegateVotesChanged(_account, previousDelegateVotes, currentDelegateVotes);
emit DelegatorBondedAmountChanged(_account, previous.bondedAmount, currentBondedAmount);

**Recommended Change:**


if (previousDelegate != newDelegate) {
    emit DelegateChanged(_account, previousDelegate, newDelegate);
}

if (previousDelegateVotes != currentDelegateVotes) {
    emit DelegateVotesChanged(_account, previousDelegateVotes, currentDelegateVotes);
}

if (previous.bondedAmount != currentBondedAmount) {
    emit DelegatorBondedAmountChanged(_account, previous.bondedAmount, currentBondedAmount);
}


## [G-06] Reduce Redundant PreciseMathUtils.percPoints(1, 1) Calls
**Location:** Inside updateCumulativeRewardFactor and delegatorCumulativeStakeAndFees functions.

**Description:** calls PreciseMathUtils.percPoints(1, 1) multiple times. we can save gas by storing this value in a constant variable and reusing it.

**Code snippet:**


if (_prevEarningsPool.cumulativeRewardFactor != 0) {
    prevCumulativeRewardFactor = _prevEarningsPool.cumulativeRewardFactor;
} else {
    prevCumulativeRewardFactor = PreciseMathUtils.percPoints(1, 1);
}

**Recommended Change:**


uint256 defaultCumulativeRewardFactor = PreciseMathUtils.percPoints(1, 1);
if (_prevEarningsPool.cumulativeRewardFactor != 0) {
    prevCumulativeRewardFactor = _prevEarningsPool.cumulativeRewardFactor;
} else {
    prevCumulativeRewardFactor = defaultCumulativeRewardFactor;
}

## [G-07] Remove Redundant Variables
**Location:** Inside the updateCumulativeFeeFactor function.

**Description:** The prevCumulativeRewardFactor is computed but not used. we can remove this variable to save gas.

 **Code snippet:**


uint256 prevCumulativeFeeFactor = _prevEarningsPool.cumulativeFeeFactor;
uint256 prevCumulativeRewardFactor = _prevEarningsPool.cumulativeRewardFactor != 0
    ? _prevEarningsPool.cumulativeRewardFactor
    : PreciseMathUtils.percPoints(1, 1);

**Recommended Change:**


uint256 prevCumulativeFeeFactor = _prevEarningsPool.cumulativeFeeFactor;

## [G-08] Optimize Binary Search
**Location:** Inside the findLowerBound function.

**Description:** The current implementation performs a binary search to find the upper bound and then adjusts the result to find the lower bound. we can optimize this by directly performing a binary search to find the lower bound, eliminating the need for adjustments.

 **Code snippet:**


uint256 upperIdx = _array.findUpperBound(_val);
// ...
return upperIdx - 1;

**Recommended Change:**


uint256 lowerIdx = _array.findLowerBound(_val);
return lowerIdx;

## [G-09] Reduce Array Length Check
**Location:** Inside the findLowerBound function.

**Description:** The current implementation checks if the array is empty at the beginning and again after finding the upper bound. we can reduce gas usage by removing the second check since it's redundant.

**Code snippet:**


uint256 len = _array.length;
if (len == 0) {
    return 0;
}
// ...
if (upperIdx == 0) {
    return len;
}

**Recommended Change:**


uint256 len = _array.length;
if (len == 0) {
    return 0;
}
// ...

## [G-10] Reduce Storage Reads
**Location:** Inside the _quorumReached and _voteSucceeded functions.

**Description:** These functions read from storage to get the proposal's vote counts multiple times. we can optimize by reading the values once and storing them in local variables to reduce storage reads.

**Code snippet:**


(uint256 againstVotes, uint256 forVotes, uint256 abstainVotes) = proposalVotes(_proposalId);

**Recommended Change:**


uint256 _proposalIdVotes = proposalVotes(_proposalId);
uint256 againstVotes = _proposalIdVotes[0];
uint256 forVotes = _proposalIdVotes[1];
uint256 abstainVotes = _proposalIdVotes[2];

## [G-11] Use Local Variable for delegate Address
**Location:** Inside the _handleVoteOverrides function.

**Description:** Instead of repeatedly calling votes().delegatedAt(_account, timepoint), we can store the result in a local variable to avoid multiple function calls.

**Code snippet:**


address delegate = votes().delegatedAt(_account, timepoint);

**Recommended Change:**


address delegate = votes().delegatedAt(_account, timepoint);