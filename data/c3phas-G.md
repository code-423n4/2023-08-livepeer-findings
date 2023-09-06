# Codebase Optimization Report

## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

## Table Of Contents

- [Codebase Optimization Report](#codebase-optimization-report)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Table Of Contents](#table-of-contents)
  - [Note on Gas estimates.](#note-on-gas-estimates)
  - [\[G-01\]Function `processRebond()` can be more optimized(Save 207 Gas on average)](#g-01function-processrebond-can-be-more-optimizedsave-207-gas-on-average)
  - [\[G-02\]Function `withdrawStake()` can be more optimized(Save 443 Gas on average)](#g-02function-withdrawstake-can-be-more-optimizedsave-443-gas-on-average)
  - [\[G-03\]Function `transcoderWithHint()` can be more optimized(Save 4749 Gas on average)](#g-03function-transcoderwithhint-can-be-more-optimizedsave-4749-gas-on-average)
  - [\[G-04\]Function `rewardWithHint()` optimization (Save 4716 Gas on average)](#g-04function-rewardwithhint-optimization-save-4716-gas-on-average)
  - [\[G-05\]Function `increaseTotalStakeUncheckpointed()` can be more optimized(save 148 Gas on average)](#g-05function-increasetotalstakeuncheckpointed-can-be-more-optimizedsave-148-gas-on-average)
  - [\[G-06\]Refactor the function `unbondWithHint()`(Save 1452 Gas)](#g-06refactor-the-function-unbondwithhintsave-1452-gas)
  - [\[G-07\]Function `slashTranscoder()` can be optimized](#g-07function-slashtranscoder-can-be-optimized)
  - [\[G-08\]BondingVotes.sol: We can optimize the function `checkpointBondingState`](#g-08bondingvotessol-we-can-optimize-the-function-checkpointbondingstate)
  - [\[G-09\]Optimizing check order for cost efficient function execution](#g-09optimizing-check-order-for-cost-efficient-function-execution)
    - [\[G-09-01\]Validate the function parameter `_amount` first before making any state reads/writes](#g-09-01validate-the-function-parameter-_amount-first-before-making-any-state-readswrites)
  - [\[G-10\]Change the model of how events are emitted](#g-10change-the-model-of-how-events-are-emitted)
    - [\[G-10-01\]BondingManager.sol.setUnbondingPeriod(): emit `_unbondingPeriod` instead of `"unbondingPeriod"`(Save 593 Gas on average)](#g-10-01bondingmanagersolsetunbondingperiod-emit-_unbondingperiod-instead-of-unbondingperiodsave-593-gas-on-average)
    - [\[G-10-02\]BondingManager.sol.setNumActiveTranscoders(): Emit `_numActiveTranscoders` instead of `"numActiveTranscoders"`(Save 584 Gas on average)](#g-10-02bondingmanagersolsetnumactivetranscoders-emit-_numactivetranscoders-instead-of-numactivetranscoderssave-584-gas-on-average)
    - [\[G-10-03\]BondingManager.sol.setTreasuryBalanceCeiling(): Emit `_ceiling` instead of `"treasuryBalanceCeiling"`](#g-10-03bondingmanagersolsettreasurybalanceceiling-emit-_ceiling-instead-of-treasurybalanceceiling)
    - [\[G-10-04\]BondingManager.sol.setCurrentRoundTotalActiveStake(): Emit `treasuryRewardCutRate` instead of `"treasuryRewardCutRate"`](#g-10-04bondingmanagersolsetcurrentroundtotalactivestake-emit-treasuryrewardcutrate-instead-of-treasuryrewardcutrate)
    - [\[G-10-05\]BondingManager.sol.\_setTreasuryRewardCutRate(): Emit `_cutRate` instead of `"nextRoundTreasuryRewardCutRate"`](#g-10-05bondingmanagersol_settreasuryrewardcutrate-emit-_cutrate-instead-of-nextroundtreasuryrewardcutrate)
  - [Conclusion](#conclusion)


## Note on Gas estimates.

we've tried to give the exact amount of gas being saved from running the included tests. whenever the function is within the test coverage

**All suggested changes have been  tested per function to ensure we don't encounter stack too deep error.**
Most of the changes involve rewriting functions and as such changes are suggested on a function basis.
We would love to look at this code again if they end up implementing the changes on this report

*when doing the refactoring, we've avoided including any changes that were reported by the bot and we encourage you to go through the bots finding to maximum the amount of gas being saved*

## [G-01]Function `processRebond()` can be more optimized(Save 207 Gas on average)

**Gas benchmarks based on function `rebondWithHint()` which calls our function of interest

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | 229425    | 231632   | 230529 
| After  | 229218    | 231425   | 230322 

```solidity
File: /contracts/bonding/BondingManager.sol
1564:    function processRebond(
1565:        address _delegator,
1566:        uint256 _unbondingLockId,
1567:        address _newPosPrev,
1568:        address _newPosNext
1569:    ) internal autoCheckpoint(_delegator) {
1570:        Delegator storage del = delegators[_delegator];
1571:        UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];

1573:        require(isValidUnbondingLock(_delegator, _unbondingLockId), "invalid unbonding lock ID");
```

Of interest to us is the require statement on line 1573
`require(isValidUnbondingLock(_delegator, _unbondingLockId), "invalid unbonding lock ID");`

The statement makes a function call to `isValidUnbondingLock()` which we can see it's implementation on https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L1167-L1170
```solidity
File: /contracts/bonding/BondingManager.sol
1167:    function isValidUnbondingLock(address _delegator, uint256 _unbondingLockId) public view returns (bool) {
1168:       // A unbonding lock is only valid if it has a non-zero withdraw round (the default value is zero)
1169:        return delegators[_delegator].unbondingLocks[_unbondingLockId].withdrawRound > 0;
1170:    }
```
Note the return statement in the above function `return delegators[_delegator].unbondingLocks[_unbondingLockId].withdrawRound > 0;`
We make some state reads by reading `delegators[_delegator]` 

Back to our original function context, note we also made the same state read on https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L1570
```solidity
1570:        Delegator storage del = delegators[_delegator];
```
The require statement ends up doing the following 
`require(delegators[_delegator].unbondingLocks[_unbondingLockId].withdrawRound > 0,"invalid unbonding lock ID");`
Note the variable `delegators[_delegator].unbondingLocks[_unbondingLockId]` is equivalent to the variable `lock` declared here  `UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];`

If we inline the function `isValidUnbondingLock()` we can avoid making this two state reads and take advantage of the already declared variable

```diff
diff --git a/contracts/bonding/BondingManager.sol b/contracts/bonding/BondingManager.sol
index 93e06ba..2482d64 100644
--- a/contracts/bonding/BondingManager.sol
+++ b/contracts/bonding/BondingManager.sol
@@ -1569,8 +1569,7 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
     ) internal autoCheckpoint(_delegator) {
         Delegator storage del = delegators[_delegator];
         UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];
-
-        require(isValidUnbondingLock(_delegator, _unbondingLockId), "invalid unbonding lock ID");
+        require(lock.withdrawRound > 0,"invalid unbonding lock ID");
```


https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L249-L257

## [G-02]Function `withdrawStake()` can be more optimized(Save 443 Gas on average)

|        | Min    | Max    | Avg    |     |
| ------ | ------ | ------ | ------ | --- |
| Before | 104389 | 121489 | 110093 |     |
| After  | 103946 | 121046 | 109650 |     |
|        |        |        |        |     |
```solidity
File: /contracts/bonding/BondingManager.sol
249:    function withdrawStake(uint256 _unbondingLockId) external whenSystemNotPaused currentRoundInitialized {
250:        Delegator storage del = delegators[msg.sender];
251:        UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];

253:        require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID");
254:        require(
255:            lock.withdrawRound <= roundsManager().currentRound(),
256:            "withdraw round must be before or equal to the current round"
257:        );
```

**Similar to the previous explanations with some additional refactoring**
After the refactoring explained in the previous case, the variable `lock.withdrawRound` was found to be called 3 times, with the 3rd call being a memory store ie `uint256 withdrawRound = lock.withdrawRound;` We can optimize the code further by making this memory store call before we make our validations using the require statements and use the memory variable in the require statement instead of reading the `lock.withdrawRound`. in the worst case(sad path) where we fail on the 1st require check, we would only waste ~6 gas for `mstore/mload`. The benefit outweigh the cons thus we should refactor.

```diff
diff --git a/contracts/bonding/BondingManager.sol b/contracts/bonding/BondingManager.sol
index 93e06ba..5f0e073 100644
--- a/contracts/bonding/BondingManager.sol
+++ b/contracts/bonding/BondingManager.sol
@@ -249,15 +249,15 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
     function withdrawStake(uint256 _unbondingLockId) external whenSystemNotPaused currentRoundInitialized {
         Delegator storage del = delegators[msg.sender];
         UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];
-
-        require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID");
+        uint256 withdrawRound = lock.withdrawRound;
+        require(withdrawRound > 0,"invalid unbonding lock ID");
         require(
-            lock.withdrawRound <= roundsManager().currentRound(),
+            withdrawRound <= roundsManager().currentRound(),
             "withdraw round must be before or equal to the current round"
         );

         uint256 amount = lock.amount;
-        uint256 withdrawRound = lock.withdrawRound;
+
         // Delete unbonding lock
         delete del.unbondingLocks[_unbondingLockId];
```



https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L485-L502

## [G-03]Function `transcoderWithHint()` can be more optimized(Save 4749 Gas on average)

|        | Min    | Max    | Avg    |     |
| ------ | ------ | ------ | ------ | --- |
| Before | - | - | 276355 |     |
| After  | - | - | 271606 |     |
|        |        |        |        |     |

```solidity
File: /contracts/bonding/BondingManager.sol
485:    function transcoderWithHint(

490:    ) public whenSystemNotPaused currentRoundInitialized {
491:        require(!roundsManager().currentRoundLocked(), "can't update transcoder params, current round is locked");
492:        require(MathUtils.validPerc(_rewardCut), "invalid rewardCut percentage");
493:        require(MathUtils.validPerc(_feeShare), "invalid feeShare percentage");
494:        require(isRegisteredTranscoder(msg.sender), "transcoder must be registered");

496:        Transcoder storage t = transcoders[msg.sender];
497:        uint256 currentRound = roundsManager().currentRound();

499:        require(
500:            !isActiveTranscoder(msg.sender) || t.lastRewardRound == currentRound,
501:            "caller can't be active or must have already called reward for the current round"
502:        );


507:        if (!transcoderPool.contains(msg.sender)) {
508:            tryToJoinActiveSet(
509:                msg.sender,
510:                delegators[msg.sender].delegatedAmount,
```


The require statement  on line 499 makes a call to the function `isActiveTranscoder(msg.sender)` which has the following implementation 
https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L1145-L1149
```solidity
File: /contracts/bonding/BondingManager.sol
1145:    function isActiveTranscoder(address _transcoder) public view returns (bool) {
1146:        Transcoder storage t = transcoders[_transcoder];
1147:        uint256 currentRound = roundsManager().currentRound();
1148:        return t.activationRound <= currentRound && currentRound < t.deactivationRound;
1149:    }
```

Note the fist two lines of this implementation 
```solidity
1146:        Transcoder storage t = transcoders[_transcoder];
1147:        uint256 currentRound = roundsManager().currentRound();
```
If we look at the calling function, `transcoderWithHint()` we are making the same calls 

```solidity
496:        Transcoder storage t = transcoders[msg.sender];
497:        uint256 currentRound = roundsManager().currentRound();
```
**Additional optimizations**
On line 494, we have a check `require(isRegisteredTranscoder(msg.sender), "transcoder must be registered");` that makes a function call to `isRegisteredTranscoder` which has the following implementation
```solidity
1156:    function isRegisteredTranscoder(address _transcoder) public view returns (bool) {
1157:        Delegator storage d = delegators[_transcoder];
1158:        return d.delegateAddress == _transcoder && d.bondedAmount > 0;
1159:    }
```
In our context we pass `msg.sender` as the value for `_transcoder`. If we could inline this function, we can avoid making another state read to `delegators[msg.sender]` on the line [510](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L510) in the main function. we read it here `delegators[msg.sender].delegatedAmount,`
Since we already cache it as `d` referencing `d` would save us some gas

```diff
diff --git a/contracts/bonding/BondingManager.sol b/contracts/bonding/BondingManager.sol
index 93e06ba..3fc436b 100644
--- a/contracts/bonding/BondingManager.sol
+++ b/contracts/bonding/BondingManager.sol
@@ -491,13 +491,12 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
         require(!roundsManager().currentRoundLocked(), "can't update transcoder params, current round is locked");
         require(MathUtils.validPerc(_rewardCut), "invalid rewardCut percentage");
         require(MathUtils.validPerc(_feeShare), "invalid feeShare percentage");
-        require(isRegisteredTranscoder(msg.sender), "transcoder must be registered");
+        Delegator storage d = delegators[msg.sender];
+        require(d.delegateAddress == msg.sender && d.bondedAmount > 0,"transcoder must be registered");

         Transcoder storage t = transcoders[msg.sender];
         uint256 currentRound = roundsManager().currentRound();
-
-        require(
-            !isActiveTranscoder(msg.sender) || t.lastRewardRound == currentRound,
+        require(!(t.activationRound <= currentRound && currentRound < t.deactivationRound) || t.lastRewardRound == currentRound,
             "caller can't be active or must have already called reward for the current round"
         );

@@ -507,7 +506,7 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
         if (!transcoderPool.contains(msg.sender)) {
             tryToJoinActiveSet(
                 msg.sender,
-                delegators[msg.sender].delegatedAmount,
+                d.delegatedAmount,
                 currentRound.add(1),
                 _newPosPrev,
                 _newPosNext
```



https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L842-L900

## [G-04]Function `rewardWithHint()` optimization (Save 4716 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | -    | 453410   | - | - |
| After  | -    | 448694   | - | - |
```solidity
File: /contracts/bonding/BondingManager.sol
842:    function rewardWithHint(address _newPosPrev, address _newPosNext)

847:    {
848:        uint256 currentRound = roundsManager().currentRound();

850:        require(isActiveTranscoder(msg.sender), "caller must be an active transcoder");
851:        require(
852:            transcoders[msg.sender].lastRewardRound != currentRound,
853:            "caller has already called reward for the current round"
854:        );

856:        Transcoder storage t = transcoders[msg.sender];
```


We look into the implementation of the function `isActiveTranscoder` which is being called in the require statement
The function has the following implementation
```solidity
1145:    function isActiveTranscoder(address _transcoder) public view returns (bool) {
1146:        Transcoder storage t = transcoders[_transcoder];
1147:        uint256 currentRound = roundsManager().currentRound();
1148:        return t.activationRound <= currentRound && currentRound < t.deactivationRound;
1149:    }
```

Put the above implementation in the context of our main function and we see two similar lines in both functions
```solidity
1146:        Transcoder storage t = transcoders[_transcoder];
1147:        uint256 currentRound = roundsManager().currentRound();
```

This means we are making this calls twice, one in the function being called, and the other one by the calling function
To avoid this, we can inline the function `isActiveTranscoder()` and refactor the code as follows

```diff
diff --git a/contracts/bonding/BondingManager.sol b/contracts/bonding/BondingManager.sol
index 93e06ba..34c2832 100644
--- a/contracts/bonding/BondingManager.sol
+++ b/contracts/bonding/BondingManager.sol
@@ -846,14 +846,13 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
         autoCheckpoint(msg.sender)
     {
         uint256 currentRound = roundsManager().currentRound();
-
-        require(isActiveTranscoder(msg.sender), "caller must be an active transcoder");
+        Transcoder storage t = transcoders[msg.sender];
+        require(t.activationRound <= currentRound && currentRound < t.deactivationRound,"caller must be an active transcoder");
         require(
-            transcoders[msg.sender].lastRewardRound != currentRound,
+            t.lastRewardRound != currentRound,
             "caller has already called reward for the current round"
         );
-
-        Transcoder storage t = transcoders[msg.sender];
+
         EarningsPool.Data storage earningsPool = t.earningsPoolPerRound[currentRound];

         // Set last round that transcoder called reward
```


https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L1307-L1345

## [G-05]Function `increaseTotalStakeUncheckpointed()` can be more optimized(save 148 Gas on average)

**Gas benchmarks based on function `rewardWithHint` that calls our internal function

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | -    | 453410   | - | - |
| After  | -    | 453262   | - | - |
```solidity
File: /contracts/bonding/BondingManager.sol
1307:    function increaseTotalStakeUncheckpointed(

1312:    ) internal {

1318:        if (isRegisteredTranscoder(_delegate)) {
1319:            uint256 currRound = roundsManager().currentRound();
1320:            uint256 nextRound = currRound.add(1);

1343:        // Increase delegate's delegated amount
1344:        delegators[_delegate].delegatedAmount = newStake;
1345:    }
```

The if statement makes a call to a function `isRegisteredTranscoder(_delegate)` whose implementation is as follows
```solidity
1156:    function isRegisteredTranscoder(address _transcoder) public view returns (bool) {
1157:        Delegator storage d = delegators[_transcoder];
1158:        return d.delegateAddress == _transcoder && d.bondedAmount > 0;
1159:    }
```
Note in the function above, we cache `delegators[_transcoder]` with `_transcoder` being the address passed to the function, so in the context of our function it would look like `Delegator storage d = delegators[_delegate];`
On the calling function line 1344, we write to the state variable `delegators[_delegate].delegatedAmount`, as we already cached the variable `delegators[_delegate]` we don't need to call it directly but instead we should use the cached reference. As the cached one is simply a pointer to the original one, we would still be writing to the state variable as required.

```diff
-        if (isRegisteredTranscoder(_delegate)) {
+        Delegator storage d = delegators[_delegate];
+        if (d.delegateAddress == _delegate && d.bondedAmount > 0) {
             uint256 currRound = roundsManager().currentRound();
             uint256 nextRound = currRound.add(1);

@@ -1341,8 +1341,7 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
         }

         // Increase delegate's delegated amount
-        delegators[_delegate].delegatedAmount = newStake;
-    }
+        d.delegatedAmount = newStake;//@audit Refactor the call due to the function call isRegisteredTranscoder?    }

```


https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L745-L784

## [G-06]Refactor the function `unbondWithHint()`(Save 1452 Gas)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | -    | 509309   | - | - |
| After  | -    | 507857   | - | - |

```solidity
File: /contracts/bonding/BondingManager.sol
745:    function unbondWithHint(

749:    ) public whenSystemNotPaused currentRoundInitialized autoClaimEarnings(msg.sender) autoCheckpoint(msg.sender) {
750:        require(delegatorStatus(msg.sender) == DelegatorStatus.Bonded, "caller must be bonded");

752:        Delegator storage del = delegators[msg.sender];
```


The require statement calls the function `delegatorStatus(msg.sender)` which has the implementation

```solidity
956:    function delegatorStatus(address _delegator) public view returns (DelegatorStatus) {
957:        Delegator storage del = delegators[_delegator];

959:        if (del.bondedAmount == 0) {
960:            // Delegator unbonded all its tokens
961:            return DelegatorStatus.Unbonded;
962:        } else if (del.startRound > roundsManager().currentRound()) {
963:            // Delegator round start is in the future
964:            return DelegatorStatus.Pending;
965:        } else {

969:            return DelegatorStatus.Bonded;
970:        }
971:    }
```

Note we make a state read `Delegator storage del = delegators[_delegator];`
In the calling function , we pass `msg.sender` as the function parameter which means our state read in the function being called is `Delegator storage del = delegators[msg.sender];` which is the same state read we make in the main function. This means we are making this call twice(too much gas)
For this to work, we introduce a new state variable to keep track of the status as shown below

```diff
diff --git a/contracts/bonding/BondingManager.sol b/contracts/bonding/BondingManager.sol
index 93e06ba..3ec3b73 100644
--- a/contracts/bonding/BondingManager.sol
+++ b/contracts/bonding/BondingManager.sol
@@ -33,6 +33,7 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {

     // Time between unbonding and possible withdrawl in rounds
     uint64 public unbondingPeriod;
+        DelegatorStatus delegatorStatus_;

     // Represents a transcoder's current state
     struct Transcoder {
@@ -742,20 +743,34 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
      * @param _newPosPrev Address of previous transcoder in pool if the caller remains in the pool
      * @param _newPosNext Address of next transcoder in pool if the caller remains in the pool
      */
+
     function unbondWithHint(
         uint256 _amount,
         address _newPosPrev,
         address _newPosNext
     ) public whenSystemNotPaused currentRoundInitialized autoClaimEarnings(msg.sender) autoCheckpoint(msg.sender) {
-        require(delegatorStatus(msg.sender) == DelegatorStatus.Bonded, "caller must be bonded");

         Delegator storage del = delegators[msg.sender];
-
         require(_amount > 0, "unbond amount must be greater than 0");
         require(_amount <= del.bondedAmount, "amount is greater than bonded amount");
+        uint256 currentRound = roundsManager().currentRound();
+        if (del.bondedAmount == 0) {
+            // Delegator unbonded all its tokens
+            delegatorStatus_ = DelegatorStatus.Unbonded;
+        } else if (del.startRound > currentRound) {
+            // Delegator unbonded all its tokens
+            delegatorStatus_ = DelegatorStatus.Pending;
+        } else {
+            // Delegator round start is now or in the past
+            // del.startRound != 0 here because if del.startRound = 0 then del.bondedAmount = 0 which
+            // would trigger the first if clause
+            delegatorStatus_ = DelegatorStatus.Bonded;
+        }
+
+        require(delegatorStatus_ == DelegatorStatus.Bonded, "caller must be bonded");

         address currentDelegate = del.delegateAddress;
-        uint256 currentRound = roundsManager().currentRound();
+
         uint256 withdrawRound = currentRound.add(unbondingPeriod);
         uint256 unbondingLockId = del.nextUnbondingLockId;

```


https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L394-L418

## [G-07]Function `slashTranscoder()` can be optimized

**Not covered by the tests**
```solidity
File: /contracts/bonding/BondingManager.sol
394:    function slashTranscoder(

399:    ) external whenSystemNotPaused onlyVerifier autoClaimEarnings(_transcoder) autoCheckpoint(_transcoder) {
400:        Delegator storage del = delegators[_transcoder];

402:        if (del.bondedAmount > 0) {
403:            uint256 penalty = MathUtils.percOf(delegators[_transcoder].bondedAmount, _slashAmount);

413:            // If still bonded decrease delegate's delegated amount
414:            if (delegatorStatus(_transcoder) == DelegatorStatus.Bonded) {
415:                delegators[del.delegateAddress].delegatedAmount = delegators[del.delegateAddress].delegatedAmount.sub(
416:                    penalty
417:                );
418:            }
```
The function call `delegatorStatus(_transcoder)` ends up making a similar state read like the one in the calling function ie  `Delegator storage del = delegators[_transcoder];`
We can refactor the code to avoid this as follows.

```diff
     // Time between unbonding and possible withdrawl in rounds
     uint64 public unbondingPeriod;
+    DelegatorStatus delegatorStatus_;

+
     function slashTranscoder(
         address _transcoder,
         address _finder,
@@ -400,7 +402,7 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
         Delegator storage del = delegators[_transcoder];

         if (del.bondedAmount > 0) {
-            uint256 penalty = MathUtils.percOf(delegators[_transcoder].bondedAmount, _slashAmount);
+            uint256 penalty = MathUtils.percOf(del.bondedAmount, _slashAmount);

             // If active transcoder, resign it
             if (transcoderPool.contains(_transcoder)) {
@@ -410,8 +412,21 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
             // Decrease bonded stake
             del.bondedAmount = del.bondedAmount.sub(penalty);

+            if (del.bondedAmount == 0) {
+                // Delegator unbonded all its tokens
+                delegatorStatus_= DelegatorStatus.Unbonded;
+            } else if (del.startRound > roundsManager().currentRound()) {
+                // Delegator round start is in the future
+                delegatorStatus_ =  DelegatorStatus.Pending;
+            } else {
+                // Delegator round start is now or in the past
+                // del.startRound != 0 here because if del.startRound = 0 then del.bondedAmount = 0 which
+                // would trigger the first if clause
+                delegatorStatus_ =  DelegatorStatus.Bonded;
+            }
+
             // If still bonded decrease delegate's delegated amount
-            if (delegatorStatus(_transcoder) == DelegatorStatus.Bonded) {
+            if (delegatorStatus_ == DelegatorStatus.Bonded) {
                 delegators[del.delegateAddress].delegatedAmount = delegators[del.delegateAddress].delegatedAmount.sub(
                     penalty
                 );
```


https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingVotes.sol#L258-L294

## [G-08]BondingVotes.sol: We can optimize the function `checkpointBondingState`

**No gas estimates as it was not covered by the tests**
```solidity
File: /contracts/bonding/BondingVotes.sol
258:    function checkpointBondingState(

266:    ) external virtual onlyBondingManager {

273:        BondingCheckpoint memory previous;
274:        if (hasCheckpoint(_account)) {
275:            previous = getBondingCheckpointAt(_account, _startRound);
276:        }

278:        BondingCheckpointsByRound storage checkpoints = bondingCheckpoints[_account];
```
The if statement on line 274 makes two function calls `hasCheckpoint(_account)` and `getBondingCheckpointAt(_account, _startRound)`
The implementation of `hasCheckpoint(_account)`  is as below
```solidity
315:    function hasCheckpoint(address _account) public view returns (bool) {
316:        return bondingCheckpoints[_account].startRounds.length > 0;
317:    }
```
we can see the return statement reads `bondingCheckpoints[_account]` which we already had cached in the main function as `checkpoints`. Inline the `hasCheckpoint` function would help us use the cached value.

The next function `getBondingCheckpointAt()` has an implementation that has the following state read 
`BondingCheckpointsByRound storage checkpoints = bondingCheckpoints[_account];` which is equivalent to the variable we cached in the main function. Inlining this  function would avoid making two state reads which are quite expensive.

We should refactor the code as follows
```diff
diff --git a/contracts/bonding/BondingVotes.sol b/contracts/bonding/BondingVotes.sol
index 2408c66..ca92f91 100644
--- a/contracts/bonding/BondingVotes.sol
+++ b/contracts/bonding/BondingVotes.sol
@@ -271,26 +271,44 @@ contract BondingVotes is ManagerProxyTarget, IBondingVotes {
         }

         BondingCheckpoint memory previous;
-        if (hasCheckpoint(_account)) {
-            previous = getBondingCheckpointAt(_account, _startRound);
-        }
-
         BondingCheckpointsByRound storage checkpoints = bondingCheckpoints[_account];
+        if (checkpoints.startRounds.length > 0){
+           if (_startRound > clock() + 1) {
+                 revert FutureLookup(_startRound, clock() + 1);
+            }
+            // Most of the time we will be calling this for a transcoder which checkpoints on every round through reward().
+            // On those cases we will have a checkpoint for exactly the round we want, so optimize for that.
+            BondingCheckpoint storage bond = checkpoints.data[_startRound];
+            if (bond.bondedAmount > 0) {
+                previous = bond;
+            }
+
+            uint256 startRoundIdx = checkpoints.startRounds.findLowerBound(_startRound);
+            if (startRoundIdx == checkpoints.startRounds.length) {
+                // No checkpoint at or before _round, so return the zero BondingCheckpoint value. This also happens if there
+                // are no checkpoints for _account. The voting power will be zero until the first checkpoint is made.
+                previous = bond;
+            }
+
+            uint256 startRound = checkpoints.startRounds[startRoundIdx];
+            previous = checkpoints.data[startRound];
+
+            }

-        BondingCheckpoint memory bond = BondingCheckpoint({
+        BondingCheckpoint memory _bond = BondingCheckpoint({
             bondedAmount: _bondedAmount,
             delegateAddress: _delegateAddress,
             delegatedAmount: _delegatedAmount,
             lastClaimRound: _lastClaimRound,
             lastRewardRound: _lastRewardRound
         });
-        checkpoints.data[_startRound] = bond;
+        checkpoints.data[_startRound] = _bond;

         // now store the startRound itself in the startRounds array to allow us
         // to find it and lookup in the above mapping
         checkpoints.startRounds.pushSorted(_startRound);

-        onBondingCheckpointChanged(_account, previous, bond);
+        onBondingCheckpointChanged(_account, previous, _bond);
     }

```

## [G-09]Optimizing check order for cost efficient function execution

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.
https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L745-L784

### [G-09-01]Validate the function parameter `_amount` first before making any state reads/writes

```solidity
File: /contracts/bonding/BondingManager.sol
745:    function unbondWithHint(

749:    ) public whenSystemNotPaused currentRoundInitialized autoClaimEarnings(msg.sender) autoCheckpoint(msg.sender) {
750:        require(delegatorStatus(msg.sender) == DelegatorStatus.Bonded, "caller must be bonded");

752:        Delegator storage del = delegators[msg.sender];

755:        require(_amount > 0, "unbond amount must be greater than 0");
```


```diff
diff --git a/contracts/bonding/BondingManager.sol b/contracts/bonding/BondingManager.sol
index 93e06ba..c43f3e3 100644
--- a/contracts/bonding/BondingManager.sol
+++ b/contracts/bonding/BondingManager.sol
@@ -747,11 +747,12 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
         address _newPosPrev,
         address _newPosNext
     ) public whenSystemNotPaused currentRoundInitialized autoClaimEarnings(msg.sender) autoCheckpoint(msg.sender) {
+        require(_amount > 0, "unbond amount must be greater than 0");
+
         require(delegatorStatus(msg.sender) == DelegatorStatus.Bonded, "caller must be bonded");

         Delegator storage del = delegators[msg.sender];

-        require(_amount > 0, "unbond amount must be greater than 0");
         require(_amount <= del.bondedAmount, "amount is greater than bonded amount");
```

## [G-10]Change the model of how events are emitted

Currently if we look at the event declaration we see something like this
`event ParameterUpdate(string param);` and the actual emit as `emit ParameterUpdate("unbondingPeriod");`
This method seems quite interesting especially for readability as we can clearly tell what the event is about. However , this method is quite expensive and an anti pattern
We can actually achieve the same readability by using named parameters for our event argument and naming the events using a descriptive name.

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L155-L159

### [G-10-01]BondingManager.sol.setUnbondingPeriod(): emit `_unbondingPeriod` instead of `"unbondingPeriod"`(Save 593 Gas on average)

|        | Min    | Max    | Avg    |     
| ------ | ------ | ------ | ------ | 
| Before | - | - | 61189 |     |
| After  | - | - | 60596 |     |

```solidity
File: /contracts/bonding/BondingManager.sol
155:    function setUnbondingPeriod(uint64 _unbondingPeriod) external onlyControllerOwner {
156:        unbondingPeriod = _unbondingPeriod;

158:        emit ParameterUpdate("unbondingPeriod");
159:    }
```

```diff
diff --git a/contracts/bonding/BondingManager.sol b/contracts/bonding/BondingManager.sol
index 93e06ba..482d4e9 100644
--- a/contracts/bonding/BondingManager.sol
+++ b/contracts/bonding/BondingManager.sol
@@ -136,6 +136,7 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
         _;
         _checkpointBondingState(_account, delegators[_account], transcoders[_account]);
     }
+    event setUnbondingPeriodNewValue(uint256 unbondingPeriod);

     /**
      * @notice BondingManager constructor. Only invokes constructor of base Manager contract with provided Controller address
@@ -155,7 +156,7 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
     function setUnbondingPeriod(uint64 _unbondingPeriod) external onlyControllerOwner {
         unbondingPeriod = _unbondingPeriod;

-        emit ParameterUpdate("unbondingPeriod");
+        emit setUnbondingPeriodNewValue(_unbondingPeriod);
     }
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L186-L190

### [G-10-02]BondingManager.sol.setNumActiveTranscoders(): Emit `_numActiveTranscoders` instead of `"numActiveTranscoders"`(Save 584 Gas on average)

|        | Min    | Max    | Avg    |    
| ------ | ------ | ------ | ------ |
| Before | 47192 | 64292 | 55742 |     |
| After  | 46608 | 63708 | 55158 |     |
   
```solidity
File: /contracts/bonding/BondingManager.sol
186:    function setNumActiveTranscoders(uint256 _numActiveTranscoders) external onlyControllerOwner {
187:        transcoderPool.setMaxSize(_numActiveTranscoders);

189:        emit ParameterUpdate("numActiveTranscoders");
190:    }
```

```diff
+            event setNumActiveTranscodersCeilingNewValue(uint256 _numActiveTranscoders);
+

     /**
      * @notice BondingManager constructor. Only invokes constructor of base Manager contract with provided Controller address
@@ -186,7 +188,7 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
     function setNumActiveTranscoders(uint256 _numActiveTranscoders) external onlyControllerOwner {
         transcoderPool.setMaxSize(_numActiveTranscoders);

-        emit ParameterUpdate("numActiveTranscoders");
+        emit setNumActiveTranscodersCeilingNewValue(_numActiveTranscoders);
     }

```

*The following are not covered by the tests thus we couldn't give an exact amount of gas being saved*

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L176-L180

### [G-10-03]BondingManager.sol.setTreasuryBalanceCeiling(): Emit `_ceiling` instead of `"treasuryBalanceCeiling"`

```solidity
File: /contracts/bonding/BondingManager.sol
176:    function setTreasuryBalanceCeiling(uint256 _ceiling) external onlyControllerOwner {
177:        treasuryBalanceCeiling = _ceiling;

179:        emit ParameterUpdate("treasuryBalanceCeiling");
180:    }
```

```diff
+        event settreasuryBalanceCeilingNewValue(uint256 treasuryBalanceCeiling);
+
     /**
      * @notice BondingManager constructor. Only invokes constructor of base Manager contract with provided Controller address
      * @dev This constructor will not initialize any state variables besides `controller`. The following setter functions
@@ -176,7 +178,7 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
     function setTreasuryBalanceCeiling(uint256 _ceiling) external onlyControllerOwner {
         treasuryBalanceCeiling = _ceiling;

-        emit ParameterUpdate("treasuryBalanceCeiling");
+        emit settreasuryBalanceCeilingNewValue(_ceiling);
```


https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L462-L469

### [G-10-04]BondingManager.sol.setCurrentRoundTotalActiveStake(): Emit `treasuryRewardCutRate` instead of `"treasuryRewardCutRate"`

```solidity
File: /contracts/bonding/BondingManager.sol
462:    function setCurrentRoundTotalActiveStake() external onlyRoundsManager {
463:        currentRoundTotalActiveStake = nextRoundTotalActiveStake;

465:        if (nextRoundTreasuryRewardCutRate != treasuryRewardCutRate) {
466:            treasuryRewardCutRate = nextRoundTreasuryRewardCutRate;
467:            // The treasury cut rate changes in a delayed fashion so we want to emit the parameter update event here
468:            emit ParameterUpdate("treasuryRewardCutRate");
469:        }
```

```diff
+   event setCurrentRoundTotalActiveStakeNewValue(uint256 treasuryRewardCutRate);

     /**
      * @notice BondingManager constructor. Only invokes constructor of base Manager contract with provided Controller address
@@ -465,7 +466,7 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {
         if (nextRoundTreasuryRewardCutRate != treasuryRewardCutRate) {
             treasuryRewardCutRate = nextRoundTreasuryRewardCutRate;
             // The treasury cut rate changes in a delayed fashion so we want to emit the parameter update event here
-            emit ParameterUpdate("treasuryRewardCutRate");
+            emit setCurrentRoundTotalActiveStakeNewValue(treasuryRewardCutRate);
         }
```


https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L1176-L1182

### [G-10-05]BondingManager.sol.\_setTreasuryRewardCutRate(): Emit `_cutRate` instead of `"nextRoundTreasuryRewardCutRate"`

```solidity
File: /contracts/bonding/BondingManager.sol
1176:    function _setTreasuryRewardCutRate(uint256 _cutRate) internal {
1177:        require(PreciseMathUtils.validPerc(_cutRate), "_cutRate is invalid precise percentage");

1179:        nextRoundTreasuryRewardCutRate = _cutRate;

1181:        emit ParameterUpdate("nextRoundTreasuryRewardCutRate");
1182:    }
```

```diff
+   event setTreasuryRewardCutRateNewValue(uint256 nextRoundTreasuryRewardCutRate);
     /**
      * @notice BondingManager constructor. Only invokes constructor of base Manager contract with provided Controller address
      * @dev This constructor will not initialize any state variables besides `controller`. The following setter functions
@@ -1178,7 +1178,7 @@ contract BondingManager is ManagerProxyTarget, IBondingManager {

         nextRoundTreasuryRewardCutRate = _cutRate;

-        emit ParameterUpdate("nextRoundTreasuryRewardCutRate");
+        emit setTreasuryRewardCutRateNewValue(_cutRate);
     }
```


## Conclusion

It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.
