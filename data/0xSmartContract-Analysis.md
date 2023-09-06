# 🛠️ Analysis - Livepeer Onchain Treasury Upgrade
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Test analysis | Test scope of the project and quality of tests |
|c) |Centralization risks | How was the risk of centralization handled in the project, what could be alternatives? |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|g) |Other recommendations | What is unique? How are the existing patterns used? |
|h) |New insights and learning from this audit | Things learned from the project |

## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-08-livepeer

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-08-livepeer#compile)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review|The [livepeer](https://livepeer.org/primer) explainer provides a high-level overview of the Project system and the describe the core components.|Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/code-423n4/2023-08-livepeer/blob/main/slither.txt)| The project already has a slither output, this output has been reviewed |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-08-livepeer/tree/main/src/test)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-08-livepeer#scope)|Top-down analysis of codes according to architectural design, IDE used: VsCode|
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-08-livepeer#areas-of-concern)||





## b) Test analysis

 What did the project do differently? ; 
- There are quality and detailed Unit and Integration tests in the project.

What could they have done better?;

- When designing the tests, I recommend modeling the project with the Threat Model, writing the tests accordingly, and documenting these threat models, this way the test coverage will be more understandable

 - For testing to be more effective, I recommend deploying the contracts to be tested on the testnet and have them tested there, closer to the scope.

- I recommend having complex tests in the test suite that include setup functions and nested operations within the call

##  c) Centralization risks 
There is no centrality risk in the project.

  


## d) Security Approach of the Project

#### What the project can add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ;
This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert.
https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this.
https://immunefi.com/


<br />

## e) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2023-08-livepeer/blob/main/bot-report.md

Especially Medium  detections in the Automated Finding Report should be taken into account;



## f) Gas Optimization
The project is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, but gas optimizations will not be a priority considering code readability and code base size

The most important part that will provide very high gas optimization in real terms; It is Struct Packaging, by changing the following struct values to uin128 and uint64, high gas optimization will be achieved, other optimizations are minimal;

```solidity
contracts\bonding\BondingManager.sol:

  38:     struct Transcoder {
  39:         uint256 lastRewardRound; // Last round that the transcoder called reward
  40:         uint256 rewardCut; // % of reward paid to transcoder by a delegator
  41:         uint256 feeShare; // % of fees paid to delegators by transcoder
  42:         mapping(uint256 => EarningsPool.Data) earningsPoolPerRound; // Mapping of round => earnings pool for the round
  43:         uint256 lastActiveStakeUpdateRound; // Round for which the stake was last updated while the transcoder is active
  44:         uint256 activationRound; // Round in which the transcoder became active - 0 if inactive
  45:         uint256 deactivationRound; // Round in which the transcoder will become inactive
  46:         uint256 activeCumulativeRewards; // The transcoder's cumulative rewards that are active in the current round
  47:         uint256 cumulativeRewards; // The transcoder's cumulative rewards (earned via the its active staked rewards and its reward cut).
  48:         uint256 cumulativeFees; // The transcoder's cumulative fees (earned via the its active staked rewards and its fee share)
  49:         uint256 lastFeeRound; // Latest round in which the transcoder received fees
  50:     }

  59:     struct Delegator {
  60:         uint256 bondedAmount; // The amount of bonded tokens
  61:         uint256 fees; // The amount of fees collected
  62:         address delegateAddress; // The address delegated to
  63:         uint256 delegatedAmount; // The amount of tokens delegated to the delegator
  64:         uint256 startRound; // The round the delegator transitions to bonded phase and is delegated to someone
  65:         uint256 lastClaimRound; // The last round during which the delegator claimed its earnings
  66:         uint256 nextUnbondingLockId; // ID for the next unbonding lock created
  67:         mapping(uint256 => UnbondingLock) unbondingLocks; // Mapping of unbonding lock ID => unbonding lock
  68:     }

  78:     struct UnbondingLock {
  79:         uint256 amount; // Amount of tokens being unbonded
  80:         uint256 withdrawRound; // Round at which unbonding period is over and tokens can be withdrawn
  81:     }
```


## g) Other recommendations


- The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns;
https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

- It is seen that the latest versions of imported important libraries such as Openzeppelin are not used in the project codes, it should be noted.
https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/

- A good model can be used to systematically assess the risk of the project, for example this modeling is recommended;
https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e


- Proposal Threshold: Review the Threshold value at regular intervals. Whether this limit is fair for large fluctuations in the value of the LPT must be evaluated.

- Staking Snapshots: Snapshots should be archived regularly and transparently. This creates a reliable record that can be referenced in future disputes.

- Voting Delay and Voting Period: Adjust voting times based on network activity and participant feedback. Too long or short periods can negatively affect participation. I suggest that the voting periods be determined according to the proposal detail, rather than being fixed.

- QUORUM and QUOTA:  Provide a clearer roadmap to indicate whether these values will change over time.

- Active Execution Model: Ensure that every proposal is executed correctly and securely with automated checks and tests. Auditing is recommended, even for the slightest proposal process

- Stake Counting: Identify why inactive Orchestrators are inactive and share this information with attendees. The Liveeper forum is a good place for this


- Bid Type Limit: Establish a clear set of criteria that determines how offer types will evolve over time and which offer types will be accepted.

- I suggest adding a DAO Constitution like the example below to your project, it will increase the reliability and health of the DAO mechanism
https://docs.tapioca.xyz/tapioca/legal/tapioca-dao-constitution


## h) New insights and learning from this audit 

- Learned blockchain modeling of a high-performance video infrastructure protocol for live and on-demand streaming

- I learned Liveeper delta Proposal mechanics
![image](https://github.com/code-423n4/2023-08-livepeer/assets/104318932/252687e7-7f52-4214-996a-1218f675e028)


### Time spent:
22 hours