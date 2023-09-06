# Gas Optimization

# Summary

| Number | Issue                                                                                  | Instances |
| :----: | :------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Using Storage instead of memory for structs/arrays saves gas                           |     5     |
| [G-02] | Use assembly to write address storage values                                           |     1     |
| [G-03] | A modifier used only once and not being inherited should be inlined to save gas        |     3     |
| [G-04] | Using bools for storage incurs overhead                                                |     6     |
| [G-05] | Do not calculate constants                                                             |     1     |
| [G-06] | Ternary operation is cheaper than if-else statement                                    |     1     |
| [G-07] | Sort Solidity operations using short-circuit mode                                      |     7     |
| [G-08] | Pre-increment and pre-decrement are cheaper than +1 ,-1                                |    18     |
| [G-09] | Use assembly for math (add, sub, mul, div)                                             |     5     |
| [G-10] | Use mappings instead of arrays                                                         |     2     |
| [G-11] | Use assembly to emit events                                                            |    26     |
| [G-12] | Use Modifiers Instead of Functions To Save Gas                                         |     1     |
| [G-13] | Internal functions not called by the contract should be removed to save deployment gas |     2     |
| [G-14] | Use `uint256(1)/uint256(2)` instead of true/false to save gas for changes              |     1     |
| [G-15] | Using delete statement can save gas                                                    |     3     |
| [G-16] | Use assembly to perform efficient back-to-back calls                                   |     6     |

## [G-01] Using Storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from
storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.

If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to
be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
File: BondingManager.sol

322        EarningsPool.Data memory prevEarningsPool = latestCumulativeFactorsPool(t, currentRound.sub(1));

1246        EarningsPool.Data memory startPool = cumulativeFactorsPool(_transcoder, _startRound);

1248        EarningsPool.Data memory endPool = latestCumulativeFactorsPool(_transcoder, _endRound);

1468        EarningsPool.Data memory prevEarningsPool = cumulativeFactorsPool(t, t.lastRewardRound);
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol

```diff
diff --git a/org.sol b/not.sol
index 4fd4a4b..2480514 100644
--- a/org.sol
+++ b/not.sol
@@ -4,4 +4,4 @@
         // LIP-36: Add fees for the current round instead of '_round'
         // https://github.com/livepeer/LIPs/issues/35#issuecomment-673659199
         EarningsPool.Data storage earningsPool = t.earningsPoolPerRound[currentRound];
-        EarningsPool.Data memory prevEarningsPool = latestCumulativeFactorsPool(t, currentRound.sub(1));
+        EarningsPool.Data storage prevEarningsPool = latestCumulativeFactorsPool(t, currentRound.sub(1));

```

```solidity
File: BondingVotes.sol

272        BondingCheckpoint memory previous;
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L273

## [G-02] Use assembly to write address storage values

```solidity
File: BondingVotes.sol

369        delegateAddress = bond.delegateAddress;

```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L369

## [G-03] A modifier used only once and not being inherited should be inlined to save gas

```solidity
File: bonding/BondingManager.sol

106    modifier onlyTicketBroker() {
106        _onlyTicketBroker();
106        _;
106    }
110
111    // Check if sender is RoundsManager
112    modifier onlyRoundsManager() {
113        _onlyRoundsManager();
114        _;
115    }
116
117    // Check if sender is Verifier
118    modifier onlyVerifier() {
119        _onlyVerifier();
120        _;
121    }
```

### These modifier is only used in these functions if possible inlined to save Gas

```solidity

302    function updateTranscoderWithFees(
303        address _transcoder,
304        uint256 _fees,
305        uint256 _round
306    ) external whenSystemNotPaused onlyTicketBroker {


462    function setCurrentRoundTotalActiveStake() external onlyRoundsManager {



394    function slashTranscoder(
395        address _transcoder,
396        address _finder,
397        uint256 _slashAmount,
398        uint256 _finderFee
399    ) external whenSystemNotPaused onlyVerifier autoClaimEarnings(_transcoder) autoCheckpoint(_transcoder) {
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L106-L121

## [G-04] Using bools for storage incurs overhead

// Booleans are more expensive than uint256 or any type that takes up a full
// word because each write operation emits an extra SLOAD to first read the
// slot's contents, replace the bits taken up by the boolean, and then write
// back. This is the compiler's defense against contract upgrades and
// pointer aliasing, and it cannot be disabled.

```solidity
File: bonding/BondingManager.sol

1272        bool isTranscoder = _delegator == delegateAddr;

```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1272

```solidity
File: bonding/BondingVotes.sol

370        bool isTranscoder = delegateAddress == _account;


398        bool isTranscoder = newDelegate == _account;
399        bool wasTranscoder = previousDelegate == _account;
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L370

```solidity
File: treasury/GovernorCountingOverridable.sol

38        bool hasVoted;

184        bool isTranscoder = _account == delegate;

```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L38

## [G-05] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas

```solidity
File: bonding/BondingManager.sol


32    uint256 constant MAX_FUTURE_ROUND = 2**256 - 1;
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L32

## [G-06] Ternary operation is cheaper than if-else statement

```solidity
File: bonding/BondingManager.sol

562            if (delegatorStatus(_owner) == DelegatorStatus.Unbonded) {
563                require(_to != _owner, "INVALID_DELEGATE");
564            } else {
565                require(currentDelegate == _to, "INVALID_DELEGATE_CHANGE");
566            }
```

```diff
require(
    delegatorStatus(_owner) == DelegatorStatus.Unbonded ? _to != _owner : currentDelegate == _to,
    delegatorStatus(_owner) == DelegatorStatus.Unbonded ? "INVALID_DELEGATE" : "INVALID_DELEGATE_CHANGE"
);

```

In this ternary expression, we first check if delegatorStatus(\_owner) is equal to DelegatorStatus.Unbonded. If it is, we evaluate \_to != \_owner and use the error message "INVALID_DELEGATE". Otherwise, if delegatorStatus(\_owner) is not equal to DelegatorStatus.Unbonded, we evaluate currentDelegate == \_to and use the error message "INVALID_DELEGATE_CHANGE"

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L562-L566

## [G-07] Sort Solidity operations using short-circuit mode  

The operators || and && apply the common short-circuiting rules. This means that in the expression f(x) || g(y), if f(x) evaluates to true, g(y) will not be evaluated even if it may have side-effects. So setting less costly function to f(x) and setting costly function to g(x) is efficient.

Use Short Circuiting rules to your advantage

```solidity
File: bonding/BondingManager.sol

343        if (prevEarningsPool.cumulativeRewardFactor == 0 && lastRewardRound == currentRound) {


559        if (msg.sender != _owner && msg.sender != l2Migrator()) {

576        } else if (currentBondedAmount > 0 && currentDelegate != _to) {

719        if (newDel.delegateAddress == address(0) && newDel.bondedAmount == 0) {

873            if (treasuryBalance >= treasuryBalanceCeiling && nextRoundTreasuryRewardCutRate > 0) {

1215        if (pool.cumulativeRewardFactor == 0 && lastRewardRound < _round) {


500            !isActiveTranscoder(msg.sender) || t.lastRewardRound == currentRound,

```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol

## [G-08] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
File: bonding/BondingManager.sol

1599        uint256 startRound = roundsManager().currentRound() + 1;

```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1599

```solidity
File: bonding/BondingVotes.sol


156        (uint256 amount, ) = getBondingStateAt(_account, clock() + 1);

168        (uint256 amount, ) = getBondingStateAt(_account, _round + 1);

182        return getTotalActiveStakeAt(clock() + 1);

195        return getTotalActiveStakeAt(_round + 1);

206        (, address delegateAddress) = getBondingStateAt(_account, clock() + 1);

219        (, address delegateAddress) = getBondingStateAt(_account, _round + 1);

267        if (_startRound != clock() + 1) {

268            revert InvalidStartRound(_startRound, clock() + 1);

326        if (_round > clock() + 1) {

327            revert FutureLookup(_round, clock() + 1);

427        if (_round > clock() + 1) {

428            revert FutureLookup(_round, clock() + 1);

```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol

```solidity
File: bonding/BondingVotes.sol

98            revert FutureLookup(_round, currentRound == 0 ? 0 : currentRound - 1);
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L98

```solidity
File: bonding/libraries/SortedArrays.sol


34        if (_array[len - 1] <= _val) {
35            return len - 1;


54        return upperIdx - 1;

68            uint256 last = array[array.length - 1];

```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol

## [G-09] Use assembly for math (add, sub, mul, div)

This is a good Example: [Resource](https://medium.com/@bloqarl/solidity-gas-optimization-tips-with-assembly-you-havent-heard-yet-1381c77ff078)

```solidity
File: BondingManager.sol

32    uint256 constant MAX_FUTURE_ROUND = 2**256 - 1;

1599        uint256 startRound = roundsManager().currentRound() + 1;

```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1599

```solidity
File: treasury/GovernorCountingOverridable.sol

110        uint256 totalVotes = againstVotes + forVotes + abstainVotes;

122        uint256 opinionatedVotes = againstVotes + forVotes;

188            return _weight - _voter.deductions;

```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L110

## [G-10] Use mappings instead of arrays

Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations, especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.

Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.

```solidity
File: bonding/BondingVotes.sol

60        uint256[] startRounds;

71        uint256[] rounds;

```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L60

## [G-11] Use assembly to emit events

We can use assembly to emit events efficiently by utilizing scratch space and the free memory pointer. This will allow us to potentially avoid memory expansion costs. Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

For example, for a generic emit event for eventSentAmountExample:

// uint256 id, uint256 value, uint256 amount
emit eventSentAmountExample(id, value, amount);

We can use the following assembly emit events:

```solidity
        assembly {
            let memptr := mload(0x40)
            mstore(0x00, calldataload(0x44))
            mstore(0x20, calldataload(0xa4))
            mstore(0x40, amount)
            log1(
                0x00,
                0x60,
                // keccak256("eventSentAmountExample(uint256,uint256,uint256)")
                0xa622cf392588fbf2cd020ff96b2f4ebd9c76d7a4bc7f3e6b2f18012312e76bc3
            )
            mstore(0x40, memptr)
        }

```

```solidity
File: bonding/BondingManager.sol

158        emit ParameterUpdate("unbondingPeriod");

179        emit ParameterUpdate("treasuryBalanceCeiling");

189        emit ParameterUpdate("numActiveTranscoders");

267        emit WithdrawStake(msg.sender, _unbondingLockId, amount, withdrawRound);

287        emit WithdrawFees(msg.sender, _recipient, _amount);

431                emit TranscoderSlashed(_transcoder, _finder, penalty, finderAmount);

436                emit TranscoderSlashed(_transcoder, address(0), penalty, 0);

439            emit TranscoderSlashed(_transcoder, _finder, 0, 0);

468            emit ParameterUpdate("treasuryRewardCutRate");

517        emit TranscoderUpdate(msg.sender, _rewardCut, _feeShare);

619        emit Bond(_to, currentDelegate, _owner, _amount, del.bondedAmount);

709        emit TransferBond(msg.sender, _delegator, oldDelUnbondingLockId, newDelUnbondingLockId, _amount);

783        emit Unbond(currentDelegate, msg.sender, unbondingLockId, _amount, withdrawRound);

889            emit TreasuryReward(msg.sender, trsry, treasuryRewards);

899        emit Reward(msg.sender, transcoderRewards);

1181        emit ParameterUpdate("nextRoundTreasuryRewardCutRate");

1420            emit TranscoderDeactivated(lastTranscoder, _activationRound);

1431        emit TranscoderActivated(_transcoder, _activationRound);

1446        emit TranscoderDeactivated(_transcoder, deactivationRound);

1541        emit EarningsClaimed(

1584        emit Rebond(del.delegateAddress, _delegator, _unbondingLockId, amount);
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol

```solidity
File: bonding/BondingVotes.sol

395            emit DelegateChanged(_account, previousDelegate, newDelegate);

404            emit DelegateVotesChanged(_account, previousDelegateVotes, currentDelegateVotes);

410            emit DelegatorBondedAmountChanged(_account, previous.bondedAmount, current.bondedAmount);

```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L395

## [G-12] Use Modifiers Instead of Functions To Save Gas

This with 0.8.9 compiler and optimization enabled. As you can see it's cheaper to deploy with a modifier, and it will save you about 30 gas. But sometimes modifiers increase code size of the contract.

```solidity
File:  contracts/bonding/BondingVotes.sol

553  function _onlyBondingManager() internal view {
554        if (msg.sender != address(bondingManager())) {
555            revert InvalidCaller(msg.sender, address(bondingManager()));
556        }
557    }
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L553-L557

## [G-13] Internal functions not called by the contract should be removed to save deployment gas

When you define an internal function in a Solidity contract, Solidity generates code for that function and includes it in the contract bytecode. If the function is not called by the contract itself or any of its external functions, then

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

64   function __GovernorCountingOverridable_init(uint256 _quota) internal onlyInitializing {

107  function _quorumReached(uint256 _proposalId) internal view virtual override returns (bool) {
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L64

## [G-14] Use `uint256(1)/uint256(2)` instead of true/false to save gas for changes

Avoids a Gsset (20000 gas) when changing from false to true, after having been true in the past

```solidity
File: GovernorCountingOverridable.sol


148   voter.hasVoted = true;
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L148

## [G-15] Using delete statement can save gas

Using delete instead of setting a struct or array to zero can save gas when clearing the entire data structure. The delete keyword is more gas-efficient because it clears the data structure in a single operation, whereas setting each individual element to zero in a loop can incur additional gas costs.

```solidity
File: bonding/BondingManager.sol


773    del.startRound = 0;

1535    t.cumulativeFees = 0;

1536    t.cumulativeRewards = 0;
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L773

## [G-16] Use assembly to perform efficient back-to-back calls

If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.

```solidity
File: /bonding/BondingManager.sol


328            earningsPool.setCommission(t.rewardCut, t.feeShare);

332                earningsPool.setStake(t.earningsPoolPerRound[lastUpdateRound].totalStake);

382        earningsPool.updateCumulativeFeeFactor(prevEarningsPool, delegatorsFees);



860        earningsPool.setCommission(t.rewardCut, t.feeShare);

868            earningsPool.setStake(t.earningsPoolPerRound[lastUpdateRound].totalStake);

882        uint256 totalRewardTokens = mtr.createReward(earningsPool.totalStake, currentRoundTotalActiveStake);


```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L328
