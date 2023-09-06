## QA-01. In case if transcoder unbonds without calling `reward` then delegators lose rewards.

### Details
In order to distribute rewards among delegators, transcoder should call `reward` function for each period.
In case if he decides to fully [unbond](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L745) without calling `reward` function, then delegators will not receive rewards for that period.

### Recommendation
When unbond transcoder, check that his last `lastRewardRound` is current round, or call `reward`.

## QA-02. BondingManager.withdrawFees doesn't create checkpoint
### Details
When `withdrawFees` function is called, [then `autoClaimEarnings` modifier is used](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L277C9-L277C26), which will update user's staked amount according to the earned rewards. 

As bonded amount has changed, that means that new checkpoint should be created, but `withdrawFees` function doesn't do that.
### Recommendation
Create checkpoint for `withdrawFees` function.

## QA-03. Slashed transcoder still has all delegators votes
### Details
When user delegate to transcoder, then he can use their votes. So amount of votes that transcoder has [is `delegatedAmount`](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L377).

It's possible that transcoder will be slashed. Then some penalty is slashed from his balance and also he is [resigned](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L407). The problem is that he still has `delegatedAmount` of votes that he can use for voting, while he is malicious, so  can do actions against protocol.
### Recommendation
Make resigned delegator use only bondedAmount as votes.

## QA-04. In case if updateTranscoderWithFees is called when transcoder has unbonded, then no one receives fee
### Details
`updateTranscoderWithFees` function is called in order to distribute fee to the transcoder and his delegators. It's possible that this function will be called not in the round where job was done, but later. So it's possible that transcoder will have to unbond during that time.

Once unbonded, then it will be [not possible to send fes](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L310) and delegators will use their fees.
### Recommendation
Do not know good solution. 