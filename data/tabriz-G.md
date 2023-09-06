## [G-01] Rearranging order of state variable declarations to pack them will save storage slots and gas
put them in follwing order
    uint64
    bool
    address
```
   uint256 startRound = del.lastClaimRound.add(1);
        address delegateAddr = del.delegateAddress;
        bool isTranscoder = _delegator == delegateAddr;
```
	
	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1270C7-L1272C56
	
## [G-02] IF’s/require() statements that check input arguments should be at the top of the function	

```
function increaseTotalStakeUncheckpointed(
        address _delegate,
        uint256 _amount,
        address _newPosPrev,
        address _newPosNext
    ) internal {
        Transcoder storage t = transcoders[_delegate];

        uint256 currStake = transcoderTotalStake(_delegate);
        uint256 newStake = currStake.add(_amount);

        if (isRegisteredTranscoder(_delegate)) {
            uint256 currRound = roundsManager().currentRound();
            uint256 nextRound = currRound.add(1);
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1307C5-L1320C50

```
 function updateDelegatorWithEarnings(
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

            if (del.delegateAddress == _delegator) {
                t.cumulativeFees = 0;
                t.cumulativeRewards = 0;
                // activeCumulativeRewards is not cleared here because the next reward() call will set it to cumulativeRewards
            }
        }
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1500

## [G-03] Replace string errors with custom errors
Gas saved per instance: Probably a few thousands for deployment + a few hundreds when the function reverts
Custom errors are cheaper both for deployment and reverts, since strings take up lots of bytes code while custom reverts just a single byte.