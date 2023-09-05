## QA-01. In case if transcoder unbonds without calling `reward` then delegators lose rewards.

### Details
In order to distribute rewards among delegators, transcoder should call `reward` function for each period.
In case if he decides to fully [unbond](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L745) without calling `reward` function, then delegators will not receive rewards for that period.

### Recommendation
When unbond transcoder, check that his last `lastRewardRound` is current round, or call `reward`.