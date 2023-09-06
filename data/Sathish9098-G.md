# GAS OPTIMIZATION

##

## [G] Using ``memory`` can be more ``gas-efficient`` when calling the same variable for the second time compared to using ``storage``.

[As per gas test](https://gist.github.com/sathishpic22/7105cfc254ce026bf5f90c7ec272f0ce) this will save around 150 - 200 GAS . 

### ``lock.withdrawRound`` called second time so ``memory`` is more gas efficient than ``storage``

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L251 

```diff
FILE: 2023-08-livepeer/contracts/bonding/BondingManager.sol

- 251: UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];
+ 251: UnbondingLock memory lock = del.unbondingLocks[_unbondingLockId];

```
##

## [G-] Structs can be packed into fewer storage slots

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

## [G-] ``IF’s/require()`` statements that check ``input arguments`` should be at the top of the function

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

###


```diff
FILE: 2023-08-livepeer/contracts/bonding/BondingManager.sol

+    uint256 withdrawRound = lock.withdrawRound;
 require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID");
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


```




 









