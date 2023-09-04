# Auther: Krace

# Overview
Due to the nature of encoding, the cost of video streaming transmission is quite high, which significantly constrains the growth of many video startups. Therefore, video infrastructure needs a more scalable and cost-effective solution to keep up with this growth.
Livepeer is designed to reduce the cost of video streaming transmission. Anyone could become an Orchestrator and contribute their computer resources in service of transcoding and distributing video, which will bring them fees. Orchestrator could also stake their LPT token to let delegators participate in this process.


# Approach
To evaluate the Livepeer web3 protocol's Solidity codebase, I followed a systematic and thorough approach. 

- Read documents with very fast forward way 
  - By reading the doc, I have an overview of the project. It can greatly help users reduce the cost of video transcoding.
  - Livepeer's token is called LPT. 
  - There are two key actors in the Livepeer network: Orchestrators and Delegators. Orchestrator contributes computer's resources to earn fee. Orchestrator could stake LPT towards Delegator, Delegator could earn a portion of fee.
  - Livepeer mints tokens approximately every 22.4 hours. 
  - Livepeer aims to maintain "participation rate" at 50%, so the inflation rate for each round varies based on "participation rate".
- Read the Readme.md 
  - The codebase is an upgrade. It creates an onchain community treasury.
  - The upgrades include LIP-91(LIP-89 & LIP-90) and LIP-92.
  - LIP-89: Livepeer Treasury will be created using the popular Governor Framework as Compound and Uniswap.
    - Any user with a staked LPT balance exceeding the Proposal Threshold could make a proposal
    - Stake amounts must be snapshotted at the start round of voting on a proposal
  - LIP-90: Funding Entity Conventions. There are two conventions
    - The treasury should be used to fund public goods in the Livepeer Ecosystem
    - Treasury proposals should be made by Special Purpose Entities, or (SPEs). End recipients should not apply directly to the Livepeer treasury for funding
  - LIP-92: Treasury Contribution Percentage
    - To solve the problem: how Livepeer Treasury should get populated. 
- Clone code and set up the test environment using yarn
- Analyzing the smart contract's logic 
  - Focus on Common Vulnerability Patterns: reentrancy attacks, integer overflow/underflow, and unauthorized access to sensitive functions or data.
  - Examined the use of external dependencies, ensuring that they were appropriately implemented and that their security implications were understood and accounted for.
  - Reviewed the contract's permissions and access control mechanisms, verifying that only authorized users had access to critical functions and data.
  - Simulated various scenarios, such as edge cases and unexpected inputs, to assess how the contract handled exceptional situations.


# Codebase quality analysis

## [BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol)
### Analysis
The `BondingManager` added three variables:
- `treasuryRewardCutRate` : the % of rewards should be routed to the treasury. When `rewardWithHint` is called, the % of rewards will be transferred into the treasury.
- `nextRoundTreasuryRewardCutRate` : next round's %
- `treasuryBalanceCeiling` : the upper limit of treasury's LPT
It also added a modifier `autoCheckpoint`, which is implemented in the contract `BondingVotes`. This modifier is used to add the corresponding round and Bond info to the `bondingCheckpoints` in `BondingVotes` whenever the info is updated.


### Problem
`isRegisteredTranscoder` and `isActiveTranscoder` don't need to use storage, you could use memory to save the gas.


## [BondingVotes.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol)
### Analysis
This contract primarily maintains a mapping called `bondingCheckpoints`, which logs every account's `BondingCheckpointsByRound`.
`BondingCheckpointsByRound` records a list of checkpoints and every chekcpoint's `BondingCheckpoint` info.


## [GovernorCountingOverridable.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol)
### Analysis
This contract is responsible for handling the votes of transcoders and delegators. Transcoders can vote with all their weight, while delegators, by default, follow the transcoder's vote. However, delegators can also cast their votes separately with the voting power they own, which will be deducted from the transcoder accordingly.



# Centralization risks
The only centralization risk in this project is in `BondingManager`, `onlyControllerOwner` allows the owner of controller to setup some sensitive parameters including `unbondingPeriod`, `nextRoundTreasuryRewardCutRate`, `treasuryBalanceCeiling` and `transcoderPool`'s max size.


# Mechanism review
This contract has two main roles: transcoder and delegator. Transcoders can "hire" many different delegators to work for them and are also required to send corresponding rewards to these delegators. Additionally, transcoders can vote on behalf of their delegators, and of course, delegators can independently manage their voting choices, with the corresponding votes deducted from the transcoder's already cast votes.


# Systemic risks
Based on my understanding of the system, I think that the default ability of a transcoder to vote on behalf of all delegators might carry some risks. If a transcoder makes malicious votes and their delegators do not promptly realize it or fail to modify their own votes, there is a possibility that malicious proposals could be accepted, resulting in certain risks.


# Time spent
20 hours

### Time spent:
15 hours