| Number | Optimization Details                                                               | Instances |
| :----: | :--------------------------------------------------------------------------------- | :-------: |
| [G-01] | Cache state variables with stack variables                                         |     9     |
| [G-02] | The result of a function call should be cached rather than re-calling the function |     2     |
| [G-03] | Using ternary operator instead of if else saves gas                                |     2     |

Total 3 issues

## [G-01] Cache state variables with stack variables

Caching of a state variable replaces each Gwarmaccess (100 gas) with a cheaper stack read.

_Total 9 instances - 1 file:_
The following instances are missed in the automated bot report

### Instance#1-9:

```solidity
File: contracts/bonding/BondingManager.sol
//@audit `lock.withdrawRound` at line 255
260:uint256 withdrawRound = lock.withdrawRound;

//@audit `delegators[_transcoder]` at line 400
403:uint256 penalty = MathUtils.percOf(delegators[_transcoder].bondedAmount, _slashAmount);

//@audit `currentRoundTotalActiveStake` at line 463
471:        bondingVotes().checkpointTotalActiveStake(currentRoundTotalActiveStake, roundsManager().currentRound());

//@audit `treasuryRewardCutRate` and `nextRoundTreasuryRewardCutRate` at line 465
466:             treasuryRewardCutRate = nextRoundTreasuryRewardCutRate;

//@audit `currPool` at line 595,596,599
600:                currPool.cumulativeFeeFactor = cumulativeFactorsPool(newDelegate, newDelegate.lastFeeRound)

//@audit `treasuryBalanceCeiling` at line 871
873:             if (treasuryBalance >= treasuryBalanceCeiling && nextRoundTreasuryRewardCutRate > 0) {

//@audit ` transcoderPool` at line 1323
1324:                transcoderPool.updateKey(_delegate, newStake, _newPosPrev, _newPosNext);

//@audit ` transcoderPool` at line 1363
1367:                transcoderPool.updateKey(_delegate, newStake, _newPosPrev, _newPosNext);

//@audit ` transcoderPool` at line 1401,1402,1416
1423:                       transcoderPool.insert(_transcoder, _totalStake, _newPosPrev, _newPosNext);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L83C4-L92C7
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L400
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L471
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L466
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L600
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L873
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1324
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1367
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1423

## [G-02] The result of a function call should be cached rather than re-calling the function

The function calls in solidity are expensive. If the same result of the same function calls are to be used several times, the result should be cached to reduce the gas consumption of repeated calls.

_Total 2 instances - 2 file:_

The following instances are missed in the automated bot report

### Instance#1:

```solidity
File: contracts/bonding/BondingManager.sol
//@audit `delegatorStatus(_owner)` at line 562
569: if (delegatorStatus(_owner) == DelegatorStatus.Unbonded) {
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L569

### Instance#2:

```solidity
File: contracts/bonding/BondingVotes.sol
//@audit `clock()` at line 267
268: revert InvalidStartRound(_startRound, clock() + 1);
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L268C12-L268C64

## [G-03] Using ternary operator instead of if-else saves gas

_Total 2 instances - 2 file:_

### Instance#1:

```solidity
File: contracts/bonding/BondingManager.sol
562:if (delegatorStatus(_owner) == DelegatorStatus.Unbonded) {
563:                require(_to != _owner, "INVALID_DELEGATE");
564:            } else {
565:                require(currentDelegate == _to, "INVALID_DELEGATE_CHANGE");
566:            }
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L562C12-L566C14

### Instance#2:

```solidity
File: contracts/bonding/BondingVotes.sol
267:if (_startRound != clock() + 1) {
268:            revert InvalidStartRound(_startRound, clock() + 1);
269:        } else if (_lastClaimRound >= _startRound) {
270:            revert FutureLastClaimRound(_lastClaimRound, _startRound - 1);
271:        }
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L267C9-L271C10
