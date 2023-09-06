# Gas 

# summary

|      | issue  | instance |
|------|--------|----------|
|[G-01]|Avoid updating storage when the value hasn't changed|1|
|[G-02]|State variables that are used multiple times in a function should be cached in stack variables|15|
|[G-03]|A modifier used only once and not being inherited should be inlined to save gas|3|
|[G-04]|Use Modifiers Instead of Functions To Save Gas|1|
|[G-05]|internal functions not called by the contract should be removed to save deployment gas|2|
|[G-06]|Use uint256(1)/uint256(2) instead of true/false to save gas for changes|1|
|[G-07]|Do not calculate constants|1|
|[G-08]|Using delete instead of setting info struct/array to 0 saves gas|3|
|[G-09]|Use assembly to perform efficient back-to-back calls|4|
|[G-10]|Uncheck arithmetics operations that can’t underflow/overflow|1|
|[G-11]|Remove the initializer modifier|1|
|[G-12]|Gas savings can be achieved by changing the model for assigning value to the structure (260 gas)|3|



## [G-01] Avoid updating storage when the value hasn't changed
Manipulating storage in solidity is gas-intensive. It can be optimized by avoiding unnecessary storage updates when the new value equals the existing value. If the old value is equal to the new value, not re-storing the value will avoid a Gsreset (2900 gas), potentially at the expense of a Gcoldsload (2100 gas) or a Gwarmaccess (100 gas).

There are 1 instances:

```solidity
File:  contracts/treasury/GovernorCountingOverridable.sol
69   quota = _quota;
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L69

## [G-02] State variables that are used multiple times in a function should be cached in stack variables
When performing multiple operations on a state variable in a function, it is recommended to cache it first. Either multiple reads or multiple writes to a state variable can save gas by caching it on the stack. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. Saves 100 gas per instance.

There are 15 instances:

```solidity
File: contracts/bonding/BondingManager.sol
1603        _delegator.bondedAmount,
            _delegator.delegateAddress,
            _delegator.delegatedAmount,
            _delegator.lastClaimRound,
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1603-L1607


`Fix code`
```solidity
 function _checkpointBondingState(
    address _owner,
    Delegator storage _delegator,
    Transcoder storage _transcoder
) internal {
    // start round refers to the round where the checkpointed stake will be active. The actual `startRound` value
    // in the delegators doesn't get updated on bond or claim earnings though, so we use currentRound() + 1
    // which is the only guaranteed round where the currently stored stake will be active.
    uint256 startRound = roundsManager().currentRound() + 1;
+    Delegator memory delegator = _delegator; // Create a local variable for _delegator in memory
    
    bondingVotes().checkpointBondingState(
        _owner,
        startRound,
+        delegator.bondedAmount,
+        delegator.delegateAddress,
+        delegator.delegatedAmount,
+        delegator.lastClaimRound,
       _transcoder.lastRewardRound
    );
}
```


```solidity
File:   contracts/bonding/BondingManager.sol
1211    pool = cumulativeFactorsPool(_transcoder, _round);

1213   uint256 lastRewardRound = _transcoder.lastRewardRound;

1216    pool.cumulativeRewardFactor = cumulativeFactorsPool(_transcoder, lastRewardRound).cumulativeRewardFactor;

1219   uint256 lastFeeRound = _transcoder.lastFeeRound;

1222    pool.cumulativeFeeFactor = cumulativeFactorsPool(_transcoder, lastFeeRound).cumulativeFeeFactor;
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1211


`Fix code`
```solidity
function latestCumulativeFactorsPool(Transcoder storage _transcoder, uint256 _round)
    internal
    view
    returns (EarningsPool.Data memory pool)
{
+    Transcoder memory transcoder = _transcoder; // Create a local variable for _transcoder in memory

+    pool = cumulativeFactorsPool(transcoder, _round);

+    uint256 lastRewardRound = transcoder.lastRewardRound;
    // Only use the cumulativeRewardFactor for lastRewardRound if lastRewardRound is before _round
    if (pool.cumulativeRewardFactor == 0 && lastRewardRound < _round) {
+        pool.cumulativeRewardFactor = cumulativeFactorsPool(transcoder, lastRewardRound).cumulativeRewardFactor;
    }

+    uint256 lastFeeRound = transcoder.lastFeeRound;
    // Only use the cumulativeFeeFactor for lastFeeRound if lastFeeRound is before _round
    if (pool.cumulativeFeeFactor == 0 && lastFeeRound < _round) {
+        pool.cumulativeFeeFactor = cumulativeFactorsPool(transcoder, lastFeeRound).cumulativeFeeFactor;
    }

    return pool;
}
```

```solidity
File:  contracts/bonding/BondingVotes.sol
465    bond.delegateAddress,
466    bond.lastClaimRound

470    bond.delegateAddress,

474   if (rewardRound < bond.lastClaimRound) {

477   return bond.bondedAmount;

483   bond.bondedAmount,
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L465

`Fix code`
```solidity
function delegatorCumulativeStakeAt(BondingCheckpoint storage bond, uint256 _round)
    internal
    view
    returns (uint256)
{
+    BondingCheckpoint memory bondingCheckpoint = bond; // Create a local variable for bond in memory

    EarningsPool.Data memory startPool = getTranscoderEarningsPoolForRound(
+        bondingCheckpoint.delegateAddress,
+        bondingCheckpoint.lastClaimRound
    );

    (uint256 rewardRound, EarningsPool.Data memory endPool) = getLastTranscoderRewardsEarningsPool(
+        bondingCheckpoint.delegateAddress,
        _round
    );

+    if (rewardRound < bondingCheckpoint.lastClaimRound) {
        // If the transcoder hasn't called reward() since the last time the delegator claimed earnings, there will be
        // no rewards to add to the delegator's stake, so we just return the originally bonded amount.
+        return bondingCheckpoint.bondedAmount;
    }

    (uint256 stakeWithRewards, ) = EarningsPoolLIP36.delegatorCumulativeStakeAndFees(
        startPool,
        endPool,
+        bondingCheckpoint.bondedAmount,
        0
    );
    return stakeWithRewards;
}
```

## [G-03] A modifier used only once and not being inherited should be inlined to save gas
When you use a modifier in Solidity, Solidity generates code to check the conditions of the modifier and execute the modified function if the conditions are met. This generated code can consume gas, especially if the modifier is used frequently or if the modified function is called multiple times.

By inlining a modifier that is used only once and not being inherited, you can eliminate the overhead of the generated code and reduce the gas cost of your contract.

```solidity
File:  contracts/bonding/BondingManager.sol
106     modifier onlyTicketBroker() {

112     modifier onlyRoundsManager() {    

118     modifier onlyVerifier() {    
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L106


## [G-04] Use Modifiers Instead of Functions To Save Gas
Example of two contracts with modifiers and internal view function:

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Inlined {
    function isNotExpired(bool _true) internal view {
        require(_true == true, "Exchange: EXPIRED");
    }
function foo(bool _test) public returns(uint){
            isNotExpired(_test);
            return 1;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Modifier {
modifier isNotExpired(bool _true) {
        require(_true == true, "Exchange: EXPIRED");
        _;
    }
function foo(bool _test) public isNotExpired(_test)returns(uint){
        return 1;
    }
}
```

Differences:

```
Deploy Modifier.sol
108727
Deploy Inlined.sol
110473
Modifier.foo
21532
Inlined.foo
21556
```

This with 0.8.9 compiler and optimization enabled. As you can see it's cheaper to deploy with a modifier, and it will save you about 30 gas. But sometimes modifiers increase code size of the contract.

```solidity
File:  contracts/bonding/BondingVotes.sol
553 function _onlyBondingManager() internal view {
        if (msg.sender != address(bondingManager())) {
            revert InvalidCaller(msg.sender, address(bondingManager()));
        }
    }
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L553-L557

## [G-05] internal functions not called by the contract should be removed to save deployment gas
When you define an internal function in a Solidity contract, Solidity generates code for that function and includes it in the contract bytecode. If the function is not called by the contract itself or any of its external functions, then 

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol
64   function __GovernorCountingOverridable_init(uint256 _quota) internal onlyInitializing {

107  function _quorumReached(uint256 _proposalId) internal view virtual override returns (bool) {    
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L64

## [G‑06] Use uint256(1)/uint256(2) instead of true/false to save gas for changes
Avoids a Gsset (20000 gas) when changing from false to true, after having been true in the past

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol
148   voter.hasVoted = true;
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L148


## [G-07] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas

```solidity
File:  contracts/bonding/BondingManager.sol
32    uint256 constant MAX_FUTURE_ROUND = 2**256 - 1;
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L32

## [G-08] Using delete instead of setting info struct/array to 0 saves gas
Using delete instead of setting a struct or array to zero can save gas when clearing the entire data structure. The delete keyword is more gas-efficient because it clears the data structure in a single operation, whereas setting each individual element to zero in a loop can incur additional gas costs.

```solidity
File:  contracts/bonding/BondingManager.sol
773    del.startRound = 0;

1535    t.cumulativeFees = 0;
1536    t.cumulativeRewards = 0;
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L773


## [G-09] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-10-use-assembly-to-perform-efficient-back-to-back-calls)


```solidity
File:  contracts/bonding/BondingManager.sol
1323     if (transcoderPool.contains(_delegate)) {
1324     transcoderPool.updateKey(_delegate, newStake, _newPosPrev, _newPosNext);


1401     if (transcoderPool.isFull()) {
1402     address lastTranscoder = transcoderPool.getLast();
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1323-L1324

## [G-10] Uncheck arithmetics operations that can’t underflow/overflow
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an [unchecked block](https://docs.soliditylang.org/en/v0.8.10/control-structures.html#checked-or-unchecked-arithmetic)

Replace this:
```
uint256 value = yield(a, b, c - totalFee(c), address(this));
```
with this:
```
unchecked {uint256 value = yield(a, b, c - totalFee(c), address(this));}
```

```solidity
File:  contracts/treasury/GovernorCountingOverridable.sol
188    return _weight - _voter.deductions;
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L188

## [G-11] Remove the initializer modifier
If we can just ensure that the initialize() function could only be called from within the constructor, we shouldn’t need to worry about it getting called again.

In the EVM, the constructor’s job is actually to return the bytecode that will live at the contract’s address. So, while inside a constructor, your address (address(this)) will be the deployment address, but there will be no bytecode at that address! So if we check address(this).code.length before the constructor has finished, even from within a delegatecall, we will get 0. So now let’s update our initialize() function to only run if we are inside a constructor:

```solidity
File:  contracts/treasury/Treasury.sol
21    ) external initializer {
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/Treasury.sol#L21

## [G-12] Gas savings can be achieved by changing the model for assigning value to the structure (260 gas)
By directly assigning the values to the individual fields of the bond struct, you can save gas compared to using the struct initialization syntax. This approach avoids the overhead of creating a new BondingCheckpoint instance and assigning the values using the initialization syntax.

```solidity
File:  contracts/bonding/BondingManager.sol
1:
706    newDel.unbondingLocks[newDelUnbondingLockId] = UnbondingLock({ amount: _amount, withdrawRound: withdrawRound });

2:
763   del.unbondingLocks[unbondingLockId] = UnbondingLock({ amount: _amount, withdrawRound: withdrawRound });
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L706

`fix code :`
```solidity
1:
newDel.unbondingLocks[newDelUnbondingLockId].amount = _amount;
newDel.unbondingLocks[newDelUnbondingLockId].withdrawRound = withdrawRound;

2:
UnbondingLock memory newUnbondingLock;
newUnbondingLock.amount = _amount;
newUnbondingLock.withdrawRound = withdrawRound;

newDel.unbondingLocks[newDelUnbondingLockId] = newUnbondingLock;
```

```solidity
File: contracts/bonding/BondingVotes.sol
280     BondingCheckpoint memory bond = BondingCheckpoint({
            bondedAmount: _bondedAmount,
            delegateAddress: _delegateAddress,
            delegatedAmount: _delegatedAmount,
            lastClaimRound: _lastClaimRound,
            lastRewardRound: _lastRewardRound
        });
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L280


`fix code :`

```solidity
BondingCheckpoint memory bond;

bond.bondedAmount = _bondedAmount;
bond.delegateAddress = _delegateAddress;
bond.delegatedAmount = _delegatedAmount;
bond.lastClaimRound = _lastClaimRound;
bond.lastRewardRound = _lastRewardRound;
```