## 1. Codebase Analysis

#### Summary
For this audit I had limited time so focused mainly on three aspects. 
- That the treasury logic has been implemented correctly. 
- That the implementation of the checkpoints logic is correctly updated everywhere & that this implementation hasn't effected the bonding logic in anyway. 
- The vote overriding for delegates is implemented correctly.

#### Checkpoint Logic
- Livepeer requires BondingManager & BondingVotes to have a shared state, the checkpoint logic is implemented to ensure that if BondingManager ever updates any of this state, a checkpointBondingState function is run in BondingVotes to also update these values in its  contract.

#### Treasury
- The new treasury implementation allows a percentage of the rewards in BondingManager be sent to a treasury contract up to a ceiling. One note I made while initially reviewing this feature is that the ceiling can be exceeded as the check in rewardWithHint to set the treasuryCutRate to 0 is done on the balance of the treasury before sending tokens. As a result there will always be 1 more set of tokens transferred after the ceiling is hit.

#### Vote Overriding
- This was a unique implementation I had not seen before that allows both a delegate and a delegator to vote. However as only 1 vote can count at a time if both do vote the delegators vote receives priority and is the one that will be counted.

## 2. Centralisation Risks

#### Input Validation for Admin Functions
- While unlikely a malicious/compromised admin could cause all withdrawals to be prevented by calling setUnbondingPeriod() with the max(uint64) value, essentially locking all tokens.
- Similarly a malicious/compromised verifier could call slashTranscoder with a slashAmount of 100% and apply a penalty of a users entire bondedAmount.

## 3. Audit Methodology/Time Spent

#### Summary
- I had roughly 3 hours per day to spend on this audit over 4 days which I did in the following manner. 

#### Day 1 (~3 Hours)
- I started by reading through all of the code/review docs/given materials.    
- Then worked on gaining understanding of protocol and create list of possible areas to focus on over the following days. These included:
	- Treasury Contributions
	- Check implementing checkpoint logic doesn't break existing behaviour. i.e. Checkpoint logic should not change the outcome of any bondingManager transactions.
	- Check no checkpoints are missed.
	- delegate vote override logic.
	- During this initial walk through I also came across the slashTranscoder function which reminded me of a recent Sherlock report I had read from the [Telcoin Audit](https://github.com/sherlock-audit/2022-11-telcoin-judging/issues/45). So I also spent some time writing up a report detailing how slashTranscoder can be similarly exploited.

#### Day 2 (~3 Hours)
- I spent my time ensuring that the checkpoint is updated at all locations. To do this I created a list of all spots in BondingManager that any of the variables updated in checkpointBondingState are modified. (i.e. bondedAmount, delegateAddress, delegatedAmount, lastClaimRound, lastRewardRound). From there I checked each function that modifies theses variables calls i_checkpointBondingState after updating the variable. It was using this method that I was able to find that withdrawFees can result in bondedAmount & lastClaimRound being assigned new values without calling i_checkpointBondingState.

(An example of the checklist I created for the variable bondedAmount)
- **Functions which modify bondedAmount:**
	- [x] slashTranscoder  (autoCheckpoint modifier)
	- [x] bondForWithHint  (calls i_checkpointBondingState directly)
	- [x] unbondWithHint   (autoCheckpoint modifier)
	- [x] processRebond    (autoCheckpoint modifier)
	- [ ] updateDelegatorWithEarnings. (no checkpoint, is called from autoClaimEarnings, transferBond)
		- [x] transferBond  (calls processRebond which has autoCheckpoint modifier)
		- [ ] autoClaimEarnings (no checkpoint)
			- [ ] withdrawFees - Doesn't update checkpoint
			- [x] slashTranscoder   (autoCheckpoint modifier)
			- [x] claimEarnings     (autoCheckpoint modifier)
			- [x] bondForWithHint   (calls i_checkpointBondingState directly)
			- [x] transferBond      (calls processRebond which has autoCheckpoint modifier)
			- [x] unbondWithHint    (autoCheckpoint modifier)
			- [x] rebondWithHint    (calls processRebond which has autoCheckpoint modifier)
			- [x] rebondFromUnbondedWithHint (calls processRebond which has autoCheckpoint modifier)

#### Day 3 (~3 Hours) 
- The next step I did was work my way through the diffs in BondingManager to check that the treasury logic has been implemented correctly and that the new checkpoint code doesn't effect any of the old bonding functionality. 

#### Day 4 (~3 Hours)
- Unfortunately I ran out of time so spent my last day doing a quick final pass and then writing up any reports/this analysis.   

Total time spent on Audit: ~12 Hours

### Time spent:
12 hours