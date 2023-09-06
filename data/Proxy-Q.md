### [Low Risk](#low-risk-1)

| Total Low Risk Issues | 3 |
|:--:|:--:|

| Count | Title | Instances |
|:--:|:-------| :--: |
| [L-01](#l-01-there-is-no-way-to-decrease-the-max-number-of-active-transcoders) | There is no way to decrease the max number of active transcoders | 1 |
| [L-02](#l-02-when-removing-transcoders-transcoderactivationround-should-be-set-to-0-for-inactive) | When removing transcoders `Transcoder.activationRound` should be set to 0 for inactive | 1 |
| [L-03](#l-03-no-function-implemented-for-setmaxearningsclaimsrounds) | No function implemented for `setMaxEarningsClaimsRounds()` | 1 |

### [Non-Critical](#non-critical-1)

| Total Non-Critical Issues | 2 |
|:--:|:--:|

| Count | Title | Instances |
|:--:|:-------| :--: |
| [NC-01](#nc-01-typos) | Typos | 3 |
| [NC-02](#nc-02-functions-need-a-more-descriptive-name) | Functions need a more descriptive name | 2 |

## Low Risk

### [L-01] There is no way to decrease the max number of active transcoders

[`setNumActiveTranscoders()`](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L182-L190) sets the maximum number of active transcoders however there is no way to decrease the number since [`setMaxSize()`](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/libraries/SortedDoublyLL.sol#L36-L44) does not allow it.

```javascript
function setNumActiveTranscoders(uint256 _numActiveTranscoders) external onlyControllerOwner {
    transcoderPool.setMaxSize(_numActiveTranscoders);

    emit ParameterUpdate("numActiveTranscoders");
}
```

```javascript
function setMaxSize(Data storage self, uint256 _size) public {
    require(_size > self.maxSize, "new max size must be greater than old max size");

    self.maxSize = _size;
}
```

### [L-02] When removing transcoders `Transcoder.activationRound` should be set to 0 for inactive

Function [`tryToJoinActiveSet()`](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1416-L1418) removes a transcoder if `transcoderPool.isFull()` however it does not set `Transcoder.activationRound` of the transcoder to 0 for inactive.
In [`Transcoder`](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L44) struct it says if a transcoder is inactive `activationRound` should be 0.

```javascript
uint256 activationRound; // Round in which the transcoder became active - 0 if inactive
```

```javascript
transcoderPool.remove(lastTranscoder);
transcoders[lastTranscoder].deactivationRound = _activationRound; // @audit Transcoder.activationRound should be 0 - for inactive
pendingNextRoundTotalActiveStake = pendingNextRoundTotalActiveStake.sub(lastStake);
```

### [L-03] No function implemented for `setMaxEarningsClaimsRounds()`

The [constructor](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L140-L148) natspec says that `setMaxEarningsClaimsRounds()` should be called to initialize state variables post-deployment. However there is no such function implemented anywhere

```javascript
/**
 * @notice BondingManager constructor. Only invokes constructor of base Manager contract with provided Controller address
 * @dev This constructor will not initialize any state variables besides `controller`. The following setter functions
 * should be used to initialize state variables post-deployment:
 * - setUnbondingPeriod()
 * - setNumActiveTranscoders()
 * - setMaxEarningsClaimsRounds()
 * @param _controller Address of Controller that this contract will be registered with
 */
```

## Non-critical

### [NC-01] Typos

- GovernorCountingOverriable.sol ( [#L35](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L35) , [#L59](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L59) ):

```javascript
    // @audit Typo - instead of "specicic" use "specific"
35: // @dev Tracks state of specicic voters in a single proposal.

    // @audit Typo - instead of "abstain" use "against"
59: // @notice The required percentage of "for" votes in relation to the total opinionated votes (for and abstain) for
```

- BondingVotes.sol ([#L65-66](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L65-L66)):

```javascript
    // @audit Typo - instead of "Notce that..." use "Notice that this is different"
65: // @dev Stores a list of checkpoints for the total active stake, queryable and mapped by round. Notce that
66: // differently from bonding checkpoints, it's only accessible on the specific round. To access the checkpoint for a
```

### [NC-02] Functions need a more descriptive name

Some functions say they set one thing but set something different and some function names are too ambiguous.

- [`setTreasuryRewardCutRate()`](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L167) and [_setTreasuryRewardCutRate()](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1176) set `nextRoundTreasuryRewardCutRate` not `treasuryRewardCutRate`

- [`transcoder()`](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L198) and [`transcoderWithHint()`](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L485) are too ambiguous
