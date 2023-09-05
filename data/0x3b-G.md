| *Issue*      | *Description*                                                                                                                                           |
|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| **[ G-01 ]** | [bondForWithHint](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L537-L623) has a useless if() statement |
| **[ G-02 ]** | Move require up                                                                                                                                         |


### [G-01] [bondForWithHint](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L537-L623) has a useless if() statement
Under [this] if statement the start round of the delegator is set to the next round. That is unneeded since the same exact thing is done in [_checkpointBondingState](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1591-L1609) a few lines bellow.
In [bondForWithHint](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L573)
```jsx
        if (delegatorStatus(_owner) == DelegatorStatus.Unbonded) {
            del.startRound = currentRound.add(1);
            ...
```
In [_checkpointBondingState](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1599)
```jsx
        uint256 startRound = roundsManager().currentRound() + 1;
        bondingVotes().checkpointBondingState(
            _owner,
            startRound,
            _delegator.bondedAmount,
            _delegator.delegateAddress,
            _delegator.delegatedAmount,
            _delegator.lastClaimRound,
            _transcoder.lastRewardRound
        );
```

### [G-02] Move require up
Under [bondForWithHint](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L537-L623) [this require statement](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L606) is checking a variable that was initialized and stetted way up int the function. If reverts it's gonna waste unnecessary gas, reverting so late into the call, and not in the beginning of it.