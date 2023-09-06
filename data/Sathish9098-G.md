# GAS OPTIMIZATION

##

## [G-1] Using ``memory`` can be more ``gas-efficient`` when calling the same variable for the second time compared to using ``storage``.

[As per gas test](https://gist.github.com/sathishpic22/7105cfc254ce026bf5f90c7ec272f0ce) this will save around 150 - 200 GAS . 

### ``lock.withdrawRound`` called second time so ``memory`` is more gas efficient than ``storage``

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L251 

```diff
FILE: 2023-08-livepeer/contracts/bonding/BondingManager.sol

- 251: UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];
+ 251: UnbondingLock memory lock = del.unbondingLocks[_unbondingLockId];

```
##

## [G-2] Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct.

### ``lastClaimRound`` can be ``uint96`` instead of ``uint256`` . Saves ``2000 GAS`` , ``1 SLOT``

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L58-L68

``uint96`` is a data type representing an unsigned integer with 96 bits, which can store values ranging from 0 to ``2^96 - 1``. While this is a relatively large range of values, whether it's ``more than enough`` to store timestamps and durations.


```diff
FILE: 2023-08-livepeer/contracts/bonding/BondingManager.sol

58: // Represents a delegator's current state
59:    struct Delegator {
60:        uint256 bondedAmount; // The amount of bonded tokens
61:        uint256 fees; // The amount of fees collected
62:        address delegateAddress; // The address delegated to
+ 65:        uint96 lastClaimRound; // The last round during which the delegator claimed its earnings
63:        uint256 delegatedAmount; // The amount of tokens delegated to the delegator
64:        uint256 startRound; // The round the delegator transitions to bonded phase and is delegated to someone
- 65:        uint256 lastClaimRound; // The last round during which the delegator claimed its earnings
66:        uint256 nextUnbondingLockId; // ID for the next unbonding lock created
67:        mapping(uint256 => UnbondingLock) unbondingLocks; // Mapping of unbonding lock ID => unbonding lock
68:    }

```

### ``lastClaimRound`` can be ``uint96`` instead of ``uint256`` . Saves ``2000 GAS`` , ``1 SLOT``

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L58-L68

``uint96`` is a data type representing an unsigned integer with 96 bits, which can store values ranging from 0 to ``2^96 - 1``. While this is a relatively large range of values, whether it's ``more than enough`` to store timestamps and durations.

```diff
FILE: 2023-08-livepeer/contracts/bonding/BondingVotes.sol

struct BondingCheckpoint {
        /**
         * @dev The amount of bonded tokens to another delegate as of the lastClaimRound.
         */
        uint256 bondedAmount;
        /**
         * @dev The address of the delegate the account is bonded to. In case of transcoders this is their own address.
         */
        address delegateAddress;
+         uint96 lastClaimRound;
        /**
         * @dev The amount of tokens delegated from delegators to this account. This is only set for transcoders, which
         * have to self-delegate first and then have tokens bonded from other delegators.
         */
        uint256 delegatedAmount;
        /**
         * @dev The last round during which the delegator claimed its earnings. This pegs the value of bondedAmount for
         * rewards calculation in {EarningsPoolLIP36-delegatorCumulativeStakeAndFees}.
         */
-         uint256 lastClaimRound;
        /**
         * @dev The last round during which the checkpointed account called {BondingManager-reward}. This is needed to
         * when calculating pending rewards for a delegator to this transcoder, to find the last earning pool available
         * for a given round. In that case we start from the delegator checkpoint and then fetch its delegate address
         * checkpoint as well to find the last earning pool.
         *
         * Notice that this is the only field that comes from the Transcoder struct in BondingManager, not Delegator.
         */
        uint256 lastRewardRound;
    }

```
##

## [G-3] ``IF’s/require()`` statements that check ``input arguments`` should be at the top of the function

FAIL CHEEPLY INSTEAD OF COSTLY 

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

### Cheaper to check the ``function parameter`` before creating ``storage variables ``

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L250-L251

```diff
FILE: 2023-08-livepeer/contracts/bonding/BondingManager.sol

+ 253:        require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID");
250:  Delegator storage del = delegators[msg.sender];
251:        UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];
252:
- 253:        require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID");
254:        require(
255:            lock.withdrawRound <= roundsManager().currentRound(),
266:            "withdraw round must be before or equal to the current round"
267:        );


+  754:        require(_amount > 0, "unbond amount must be greater than 0");
750: require(delegatorStatus(msg.sender) == DelegatorStatus.Bonded, "caller must be bonded");
751:
752:        Delegator storage del = delegators[msg.sender];
753:
- 754:        require(_amount > 0, "unbond amount must be greater than 0");
755:        require(_amount <= del.bondedAmount, "amount is greater than bonded amount");

```

##

## [G-4] State variables that are used multiple times in a function should be cached in stack variables

Caching state variable with memory saves ``100 GAS`` , ``1 SLOD ``

### lock.withdrawRound , del.bondedAmount , del.delegateAddress , t.cumulativeRewards  should be cached : Saves ``900 GAS`` , ``9 SLOD``

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L255

```diff
FILE: 2023-08-livepeer/contracts/bonding/BondingManager.sol

+    uint256 withdrawRound = lock.withdrawRound;
253: require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID");
        require(
            withdrawRound <= roundsManager().currentRound(),
            "withdraw round must be before or equal to the current round"
        );

        uint256 amount = lock.amount;
-        uint256 withdrawRound = lock.withdrawRound;
        // Delete unbonding lock
        delete del.unbondingLocks[_unbondingLockId];

        // Tell Minter to transfer stake (LPT) to the delegator
        minter().trustedTransferTokens(msg.sender, amount);

        emit WithdrawStake(msg.sender, _unbondingLockId, amount, withdrawRound);

+ uint256 bondedAmount_ = del.bondedAmount ;
- if (del.bondedAmount > 0) {
+ if (bondedAmount_ > 0) {
            uint256 penalty = MathUtils.percOf(delegators[_transcoder].bondedAmount, _slashAmount);

            // If active transcoder, resign it
            if (transcoderPool.contains(_transcoder)) {
                resignTranscoder(_transcoder);
            }

            // Decrease bonded stake
-             del.bondedAmount = del.bondedAmount.sub(penalty);
+             del.bondedAmount = bondedAmount_.sub(penalty);

+  uint256 delegateAddress_ = del.delegateAddress ;
- 415:  delegators[del.delegateAddress].delegatedAmount = delegators[del.delegateAddress].delegatedAmount.sub(
+ 415:  delegators[delegateAddress_ ].delegatedAmount = delegators[delegateAddress_].delegatedAmount.sub(
416:                    penalty
417:                );


+ uint256 cumulativeRewards_ = t.cumulativeRewards;
-  t.activeCumulativeRewards = t.cumulativeRewards;
+  t.activeCumulativeRewards = cumulativeRewards_ ;


        uint256 transcoderCommissionRewards = MathUtils.percOf(_rewards, earningsPool.transcoderRewardCut);
        uint256 delegatorsRewards = _rewards.sub(transcoderCommissionRewards);
        // Calculate the rewards earned by the transcoder's earned rewards
        uint256 transcoderRewardStakeRewards = PreciseMathUtils.percOf(
            delegatorsRewards,
            t.activeCumulativeRewards,
            earningsPool.totalStake
        );
        // Track rewards earned by the transcoder based on its earned rewards and rewardCut
-         t.cumulativeRewards = t.cumulativeRewards.add(transcoderRewardStakeRewards).add(transcoderCommissionRewards);
+         t.cumulativeRewards = cumulativeRewards_ .add(transcoderRewardStakeRewards).add(transcoderCommissionRewards);
   

+  address delegateAddress_ = del.delegateAddress ;
        // Only will have earnings to claim if you have a delegate
        // If not delegated, skip the earnings claim process
-        if (del.delegateAddress != address(0)) {
+        if (delegateAddress_ != address(0)) {
            (currentBondedAmount, currentFees) = pendingStakeAndFees(_delegator, _endRound);

            // Check whether the endEarningsPool is initialised
            // If it is not initialised set it's cumulative factors so that they can be used when a delegator
            // next claims earnings as the start cumulative factors (see delegatorCumulativeStakeAndFees())
-            Transcoder storage t = transcoders[del.delegateAddress];
+            Transcoder storage t = transcoders[delegateAddress_ ];
            EarningsPool.Data storage endEarningsPool = t.earningsPoolPerRound[_endRound];
            if (endEarningsPool.cumulativeRewardFactor == 0) {
                uint256 lastRewardRound = t.lastRewardRound;
                if (lastRewardRound < _endRound) {
                    endEarningsPool.cumulativeRewardFactor = cumulativeFactorsPool(t, lastRewardRound)
                        .cumulativeRewardFactor;
                }
            }
            if (endEarningsPool.cumulativeFeeFactor == 0) {
                uint256 lastFeeRound = t.lastFeeRound;
                if (lastFeeRound < _endRound) {
                    endEarningsPool.cumulativeFeeFactor = cumulativeFactorsPool(t, lastFeeRound).cumulativeFeeFactor;
                }
            }

-            if (del.delegateAddress == _delegator) {
+            if (delegateAddress_  == _delegator) {
                t.cumulativeFees = 0;
                t.cumulativeRewards = 0;
                // activeCumulativeRewards is not cleared here because the next reward() call will set it to cumulativeRewards
            }
        }

        emit EarningsClaimed(
-            del.delegateAddress,
+            delegateAddress_ ,
            _delegator,
            currentBondedAmount.sub(del.bondedAmount),
            currentFees.sub(del.fees),
            startRound,
            _endRound
        );

+  address delegateAddress_ = del.delegateAddress ;
- 1582: increaseTotalStake(del.delegateAddress, amount, _newPosPrev, _newPosNext);
+ 1582: increaseTotalStake(delegateAddress_, amount, _newPosPrev, _newPosNext);

-        emit Rebond(del.delegateAddress, _delegator, _unbondingLockId, amount);
+        emit Rebond(delegateAddress_ , _delegator, _unbondingLockId, amount);

```

### ``bond.delegateAddress`` , ``bond.lastClaimRound`` , ``bond.bondedAmount`` should be cached : Saves ``300 GAS`` , ``3 SLOD``

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingVotes.sol#L464-L487

```diff
FILE: Breadcrumbs2023-08-livepeer/contracts/bonding/BondingVotes.sol

+ address delegateAddress_ = bond.delegateAddress ;
+ uint256 lastClaimRound_ = bond.lastClaimRound ;
+ uint256 bondedAmount_ = bond.bondedAmount ;
464:  EarningsPool.Data memory startPool = getTranscoderEarningsPoolForRound(
-            bond.delegateAddress,
+            delegateAddress_,
-            bond.lastClaimRound
+            delegateAddress_
        );

        (uint256 rewardRound, EarningsPool.Data memory endPool) = getLastTranscoderRewardsEarningsPool(
-            bond.delegateAddress,
+            delegateAddress_,
            _round
        );

-  if (rewardRound < bond.lastClaimRound) {
+  if (rewardRound < lastClaimRound_ ) {
 // If the transcoder hasn't called reward() since the last time the delegator claimed earnings, there wil be
            // no rewards to add to the delegator's stake so we just return the originally bonded amount.
-            return bond.bondedAmount;
+            return bondedAmount_;
        }

(uint256 stakeWithRewards, ) = EarningsPoolLIP36.delegatorCumulativeStakeAndFees(
            startPool,
            endPool,
-            bond.bondedAmount,
+            bondedAmount_,
            0
        );

```
##

## [G-5] 




 









