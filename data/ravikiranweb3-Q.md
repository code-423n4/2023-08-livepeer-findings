1) BondingVotes contract
 a:- getLastTranscoderRewardsEarningsPool reading checkpoint in storage is not necessary.It can have be 
     memory and would be cost efficient.
 b:- getBondingCheckpointAt reading BondingCheckpointsByRound and BondingCheckpoint into storage 
     variables is not necessary, can be into memory variable and would be cost efficient
 c:- getBondingCheckpointAt - it does not look like there is a need to return BondingCheckPoint as 
     storage for this function, It can be memory and will be more cost efficient

 d: - getTotalActiveStakeAt does not required the initializedRounds to be read in storage. 


2) BondingManager contract
 a:- delegatorStatus function does not require the delegator to be loaded in storage. It could be in memory
 b:- getTranscoder function does not require the Transcoder to be loaded in storage. It could be in memory.
 c:- getTranscoderEarningsPoolForRound function does not require the EarningsPool.Data to be loaded in storage. it could be in memory.
 d:- getDelegator function does not require the Delegator to be loaded in storage. it could be in memory.
 e:- getDelegatorUnbondingLock function does not require the UnbondingLock to be loaded in storage. it could be in memory.
 f:- isActiveTranscoder function does not require the Transcoder to be loaded in storage. it could be in memory.
 g: isRegisteredTranscoder function does not require the Delegator to be loaded in storage. it could be in memory.