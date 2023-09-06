## Gas Optimizations
| Number |Issue|Instances|
|-|:-|:-:|
| [G-01] | `Structs` can be packed into fewer storage slots. | 5 | 
| [G-02] | State variables can be packed into fewer storage slots. | 1 |
| [G-03] | State variables can be cached instead of re-reading them from storage. | 3 |

## [G-01] Structs can be packed into fewer storage slots.

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs), we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).


### Reduce uint type for `lastRewardRound` , `rewardCut` , `feeShare` , `lastActiveStakeUpdateRound` , `activationRound` , `deactivationRound` and `lastFeeRound` and rearrange them in struct to save 5 SLOT (~10000 Gas)

These round holding 5 variables( `lastRewardRound` , `lastActiveStakeUpdateRound`, `activationRound` , `deactivationRound` and `lastFeeRound`) can be reduced to `uint64` because they are based on arbitrum `block.number` and `uint64` is big enough to hold them. 

`rewardCut` , `feeShare`  are holding *%* value with 1e6 precision. And max percent can be *100%* so `uint64` will be much more enough to hold them. Can be packed in `lastRewardRound` slot whose size is also truncated. 

```solidity
File : contracts/bonding/BondingManager.sol

38: struct Transcoder {
        uint256 lastRewardRound; // Last round that the transcoder called reward
        uint256 rewardCut; // % of reward paid to transcoder by a delegator
        uint256 feeShare; // % of fees paid to delegators by transcoder
        mapping(uint256 => EarningsPool.Data) earningsPoolPerRound; // Mapping of round => earnings pool for the round
        uint256 lastActiveStakeUpdateRound; // Round for which the stake was last updated while the transcoder is active
        uint256 activationRound; // Round in which the transcoder became active - 0 if inactive
        uint256 deactivationRound; // Round in which the transcoder will become inactive
        uint256 activeCumulativeRewards; // The transcoder's cumulative rewards that are active in the current round
        uint256 cumulativeRewards; // The transcoder's cumulative rewards (earned via the its active staked rewards and its reward cut).
        uint256 cumulativeFees; // The transcoder's cumulative fees (earned via the its active staked rewards and its fee share)
50:     uint256 lastFeeRound; // Latest round in which the transcoder received fees
    }

```
[38-50](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L38C5-L50C6)

```diff
File : contracts/bonding/BondingManager.sol

 struct Transcoder {
-       uint256 lastRewardRound; // Last round that the transcoder called reward
-       uint256 rewardCut; // % of reward paid to transcoder by a delegator
-       uint256 feeShare; // % of fees paid to delegators by transcoder
        mapping(uint256 => EarningsPool.Data) earningsPoolPerRound; // Mapping of round => earnings pool for the round
-       uint256 lastActiveStakeUpdateRound; // Round for which the stake was last updated while the transcoder is active
-       uint256 activationRound; // Round in which the transcoder became active - 0 if inactive
-       uint256 deactivationRound; // Round in which the transcoder will become inactive
        uint256 activeCumulativeRewards; // The transcoder's cumulative rewards that are active in the current round
        uint256 cumulativeRewards; // The transcoder's cumulative rewards (earned via the its active staked rewards and its reward cut).
        uint256 cumulativeFees; // The transcoder's cumulative fees (earned via the its active staked rewards and its fee share)
-       uint256 lastFeeRound; // Latest round in which the transcoder received fees
 
+      uint64 lastRewardRound; // Last round that the transcoder called reward
+      uint64 rewardCut; // % of reward paid to transcoder by a delegator
+      uint64 feeShare; // % of fees paid to delegators by transcoder

+      uint64 lastActiveStakeUpdateRound; // Round for which the stake was last updated while the transcoder is active
+      uint64 activationRound; // Round in which the transcoder became active - 0 if inactive
+      uint64 deactivationRound; // Round in which the transcoder will become inactive
+      uint64 lastFeeRound; // Latest round in which the transcoder received fees

    }

```

### Reduce uint type for `startRound` and `lastClaimRound` to save 1 SLOT (~2000 Gas)

Same as above round holding variables can be reduced to `uint64`.

```solidity
File : contracts/bonding/BondingManager.sol

59: struct Delegator {
        uint256 bondedAmount; // The amount of bonded tokens
        uint256 fees; // The amount of fees collected
        address delegateAddress; // The address delegated to
        uint256 delegatedAmount; // The amount of tokens delegated to the delegator
        uint256 startRound; // The round the delegator transitions to bonded phase and is delegated to someone
        uint256 lastClaimRound; // The last round during which the delegator claimed its earnings
        uint256 nextUnbondingLockId; // ID for the next unbonding lock created
        mapping(uint256 => UnbondingLock) unbondingLocks; // Mapping of unbonding lock ID => unbonding lock
68:    }

```
[59-68](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L59C5-L68C6)

```diff
File : contracts/bonding/BondingManager.sol

 struct Delegator {
        uint256 bondedAmount; // The amount of bonded tokens
        uint256 fees; // The amount of fees collected
        address delegateAddress; // The address delegated to
        uint256 delegatedAmount; // The amount of tokens delegated to the delegator
-       uint256 startRound; // The round the delegator transitions to bonded phase and is delegated to someone
-       uint256 lastClaimRound; // The last round during which the delegator claimed its earnings
+       uint64 startRound; // The round the delegator transitions to bonded phase and is delegated to someone
+       uint64 lastClaimRound; // The last round during which the delegator claimed its earnings
        uint256 nextUnbondingLockId; // ID for the next unbonding lock created
        mapping(uint256 => UnbondingLock) unbondingLocks; // Mapping of unbonding lock ID => unbonding lock
    }

```
### Reduce uint type for `lastClaimRound` and `lastRewardRound` to save 1 SLOT (~2000 Gas)
```solidity
File : contracts/bonding/BondingVotes.sol

24:  struct BondingCheckpoint {
        /**
         * @dev The amount of bonded tokens to another delegate as of the lastClaimRound.
         */
        uint256 bondedAmount;
        /**
         * @dev The address of the delegate the account is bonded to. In case of transcoders this is their own address.
         */
        address delegateAddress;
        /**
         * @dev The amount of tokens delegated from delegators to this account. This is only set for transcoders, which
         * have to self-delegate first and then have tokens bonded from other delegators.
         */
        uint256 delegatedAmount;
        /**
         * @dev The last round during which the delegator claimed its earnings. This pegs the value of bondedAmount for
         * rewards calculation in {EarningsPoolLIP36-delegatorCumulativeStakeAndFees}.
         */
        uint256 lastClaimRound;
        /**
         * @dev The last round during which the checkpointed account called {BondingManager-reward}. This is needed to
         * when calculating pending rewards for a delegator to this transcoder, to find the last earning pool available
         * for a given round. In that case we start from the delegator checkpoint and then fetch its delegate address
         * checkpoint as well to find the last earning pool.
         *
         * Notice that this is the only field that comes from the Transcoder struct in BondingManager, not Delegator.
         */
        uint256 lastRewardRound;
53:    }

```
[24-53](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L24C5-L53C1)

```diff
File : contracts/bonding/BondingVotes.sol

 struct BondingCheckpoint {
        
        uint256 bondedAmount;
      
        address delegateAddress;
        
        uint256 delegatedAmount;
       
-       uint256 lastClaimRound;
        
-       uint256 lastRewardRound;

+       uint64 lastClaimRound;
        
+       uint64 lastRewardRound;
    }

```

### Reduce uint type for `startRounds` to save 1 SLOT (~2000 Gas)
```solidity
File : contracts/bonding/BondingVotes.sol

59:  struct BondingCheckpointsByRound {
60:        uint256[] startRounds;
61:        mapping(uint256 => BondingCheckpoint) data;
62:    }

```
[59-62](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L59C4-L62C6)

```diff
File : contracts/bonding/BondingVotes.sol

59:  struct BondingCheckpointsByRound {
-60:       uint256[] startRounds;
+60:       uint64[] startRounds;
 61:        mapping(uint256 => BondingCheckpoint) data;
 62:    }

```

### Reduce uint type for `rounds` to save 1 SLOT (~2000 Gas)
```solidity
File : contracts/bonding/BondingVotes.so

70: struct TotalActiveStakeByRound {
        uint256[] rounds;
        mapping(uint256 => uint256) data;
73:    }

```
[70-73](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L70C5-L73C6)

```diff
File : contracts/bonding/BondingVotes.so

70: struct TotalActiveStakeByRound {
-       uint256[] rounds;
+       uint64[] rounds;
        mapping(uint256 => uint256) data;
73:    }

```
*Note: Change in code according to these reduced sizes. Use typecasting to store params in them and use uint64 to cache them*

## [G-02] State variables can be packed into fewer storage slots.

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions, we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper

```solidity
File : contracts/bonding/BondingManager.sol

98: uint256 public treasuryRewardCutRate;
99:    // The value for `treasuryRewardCutRate` to be set on the next round initialization.
100:    uint256 public nextRoundTreasuryRewardCutRate;

```
[98-100](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L98C4-L100C51)

These `treasuryRewardCutRate` and `nextRoundTreasuryRewardCutRate` sizes can be reduced to `uint128` because they hold *%* value with 27 Precision. And Max *%* value will be *100%* so `uin128` will be big enough to hold them. It saves 1 slot.

```diff
File : contracts/bonding/BondingManager.sol

-   uint256 public treasuryRewardCutRate;
    // The value for `treasuryRewardCutRate` to be set on the next round initialization.
-   uint256 public nextRoundTreasuryRewardCutRate;

+   uint128 public treasuryRewardCutRate;
    // The value for `treasuryRewardCutRate` to be set on the next round initialization.
+   uint128 public nextRoundTreasuryRewardCutRate;

```

## [G-03] State variables can be cached instead of re-reading them from storage.

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

### ### Cache `del.delegateAddress` to save 3 SLOAD

```solidity
File : contracts/bonding/BondingManager.sol

1500:     function updateDelegatorWithEarnings(
        address _delegator,
        uint256 _endRound,
        uint256 _lastClaimRound
    ) internal {
        Delegator storage del = delegators[_delegator];
        uint256 startRound = _lastClaimRound.add(1);
        uint256 currentBondedAmount = del.bondedAmount;
        uint256 currentFees = del.fees;

        // Only will have earnings to claim if you have a delegate
        // If not delegated, skip the earnings claim process
        if (del.delegateAddress != address(0)) {
            (currentBondedAmount, currentFees) = pendingStakeAndFees(_delegator, _endRound);

            // Check whether the endEarningsPool is initialised
            // If it is not initialised set it's cumulative factors so that they can be used when a delegator
            // next claims earnings as the start cumulative factors (see delegatorCumulativeStakeAndFees())
            Transcoder storage t = transcoders[del.delegateAddress];
            EarningsPool.Data storage endEarningsPool = t.earningsPoolPerRound[_endRound];

            if (del.delegateAddress == _delegator) {
                t.cumulativeFees = 0;
                t.cumulativeRewards = 0;
                // activeCumulativeRewards is not cleared here because the next reward() call will set it to cumulativeRewards
            }
        }

        emit EarningsClaimed(
            del.delegateAddress,
            _delegator,
            currentBondedAmount.sub(del.bondedAmount),
            currentFees.sub(del.fees),
            startRound,
            _endRound
1548:        );

```
[1500-1548](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1500C1-L1548C11)

```diff
File : contracts/bonding/BondingManager.sol

  function updateDelegatorWithEarnings(
        address _delegator,
        uint256 _endRound,
        uint256 _lastClaimRound
    ) internal {
        Delegator storage del = delegators[_delegator];
        uint256 startRound = _lastClaimRound.add(1);
        uint256 currentBondedAmount = del.bondedAmount;
        uint256 currentFees = del.fees;
+       address cached_delegate_address = del.delegateAddress;

        // Only will have earnings to claim if you have a delegate
        // If not delegated, skip the earnings claim process
-       if (del.delegateAddress != address(0)) {
+       if (cached_delegate_address != address(0)) {
            (currentBondedAmount, currentFees) = pendingStakeAndFees(_delegator, _endRound);

-           Transcoder storage t = transcoders[del.delegateAddress];
+           Transcoder storage t = transcoders[cached_delegate_address];
            EarningsPool.Data storage endEarningsPool = t.earningsPoolPerRound[_endRound];
    

-           if (del.delegateAddress == _delegator) {
+           if (cached_delegate_address == _delegator) {
                t.cumulativeFees = 0;
                t.cumulativeRewards = 0;
                // activeCumulativeRewards is not cleared here because the next reward() call will set it to cumulativeRewards
            }
        }

        emit EarningsClaimed(
-           del.delegateAddress,
+           cached_delegate_address,
            _delegator,
            currentBondedAmount.sub(del.bondedAmount),
            currentFees.sub(del.fees),
            startRound,
            _endRound
        );

```

### Cache `del.delegateAddress` to save 1 SLOAD
```solidity
File : contracts/bonding/BondingManager.sol

    function processRebond
    ...
1582:   increaseTotalStake(del.delegateAddress, amount, _newPosPrev, _newPosNext);

1584:   emit Rebond(del.delegateAddress, _delegator, _unbondingLockId, amount);

```
[1582-1584](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1582C1-L1584C80)

```diff
File : contracts/bonding/BondingManager.sol

    function processRebond
    ...
+       address cached_delegate_address = del.delegateAddress;

-1582:   increaseTotalStake(del.delegateAddress, amount, _newPosPrev, _newPosNext);
+1582:   increaseTotalStake(cached_delegate_address, amount, _newPosPrev, _newPosNext);

-1584:   emit Rebond(del.delegateAddress, _delegator, _unbondingLockId, amount);
+1584:   emit Rebond(cached_delegate_address, _delegator, _unbondingLockId, amount);

```