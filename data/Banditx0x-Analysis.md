## Audit approach

My audit approach focused on:
- Vote manipulation
- Checkpointing for all important state changes
- Unexpected inputs in any of the external non-accessed contracts in `BondingManager.sol`

I paid special attention to the code which was updated in the latest pull request, such as the `BondingVotes` contract.

I found vulnerabilities in all the above categories.

One mental mode I used for the code was based the "Assets, Actors and Actions" model from Secureum. Below are some notes I collected on **Actors** and the **Actions** they can take. Actors are in bold and actions are in dot points in the codebase Analysis:

## Codebase Analysis


**Transcoders:**
- Can vote on behalf of delegate
- Change RewardFee and RewardCut
- Collect Rewards

**Rounds Manager:**
- initialize round
- set the round length of current and future rounds
- Set lock. The lock restricts when transcoder can change their rewardFee and rewardCut

 **Verifier:**
- Slash Transcoder

**Delegator:**
- Delegate tokens to transcoder to get share of their rewards
- Unbond and rebond
- Claim rewards that have been collected by the transcoder they are delegated to

**Any User:**
- Become Transcoder
- Become Delegator
- Bond for another unbonded address. If they increase the amount bonded they have to transfer to mintr.

### Assets:

The relevant assets in this audit are votes and tokens. Although votes are not technically an asset, incorrect flow of assets can result in security vulnerabilities.

## What is new with the update:


### Checkpointing
- Records information to determine voting power
- Checkpoints delegate/transcoder information
- Checkpoints totalActiveStake

The logic for checkpointing is in the `BondingVotes.sol` contract. These checkpoints are then collected in the functions in `BondingManager.sol`, mainly using the modifier `autoCheckpoint`. This modifier runs after the function logic has executed, which makes sure that the checkpoint records the state after all state changes have occurred.

### Treasury Fees

- This is deducted during when transcoder calls fees.
- When treasury fees reach the ceiling, the fees are reduced to zero for the next round.

### Centralisation and Systemic Risks

- L2Migrator can change delegator's delegateAddress
- Verifier can slash transcoder's stake

### How much time did you spend?

I spent around 40 hours on this audit


### Time spent:
40 hours