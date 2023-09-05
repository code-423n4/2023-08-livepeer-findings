1) BondingVotes contract
 a:- getLastTranscoderRewardsEarningsPool reading checkpoint in storage is not necessary.It can have be 
     memory and would be cost efficient.
 b:- getBondingCheckpointAt reading BondingCheckpointsByRound and BondingCheckpoint into storage 
     variables is not necessary, can be into memory variable and would be cost efficient
 c:- getBondingCheckpointAt - it does not look like there is a need to return BondingCheckPoint as 
     storage for this function, It can be memory and will be more cost efficient

 d: - getTotalActiveStakeAt does not required the initializedRounds to be read in storage. 