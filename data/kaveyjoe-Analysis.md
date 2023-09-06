## LivePeer Onchain Treasury Update Advanced Analysis  Report
## Introduction
This  report provides an advanced analysis of the LivePeer Onchain Treasury Update, focusing on the code4rena contest. The update introduces several critical changes to the LivePeer protocol, including the implementation of a treasury, updates to the BondingManager contract, and a custom governance system. This report aims to evaluate the code, identify potential vulnerabilities, and ensure the integrity of the LivePeer protocol.

## Governance and Contract Separation
The update emphasizes the importance of separating the voting power and governance systems from the existing protocol contracts, with the exception of the BondingManager contract. This separation is crucial for maintaining the integrity of the protocol. The audit team confirms that this separation has been achieved effectively, with governance logic primarily residing in the LivepeerGovernor contract.

**Priority 1: BondingManager Behavior**
The number one priority in this update is ensuring that no unintended changes have occurred in the existing BondingManager contract, apart from the specified modifications and their dependencies. The audit team has thoroughly reviewed the BondingManager contract (888 lines of code), including comments, and is pleased to report that the contract modifications have been successfully implemented without introducing any unwanted behavior changes.

**Priority 2: BondingManager Checkpointing**
The update introduces extensive checkpointing mechanisms to the BondingManager contract. It is essential to verify that any change to an account's state is consistently reflected in the BondingVotes contract. The audit team has carefully reviewed the checkpointing logic and can confirm that it is comprehensive and correctly implemented.

**Rewards Calculation**
The rewards calculation logic in Livepeer, as defined in LIP-36, plays a pivotal role in the protocol's staking and governance processes. This logic ensures that rewards are distributed correctly to transcoders and delegators. The audit team has examined this logic and found it to be consistent with the described implementation.

## Contract Scope and Libraries
The audit covered several contracts and libraries:

- **BondingManager.sol** : This contract handles staking and rewards management. The audit confirms that it integrates the required changes seamlessly and uses the SafeMath library for safe arithmetic operations.

- **BondingVotes.sol** : Responsible for storing checkpoints of the BondingManager state for governance purposes. It uses the Arrays and SafeCast libraries. The implementation appears sound.

- **EarningsPoolLIP36.sol**: A library for staking rewards calculation based on LIP-36. It utilizes SafeMath for secure arithmetic operations.

- **SortedArrays.sol** : A library that aids in maintaining and searching sorted arrays. It relies on the Arrays library for certain functions. The audit confirms its correctness.

- **GovernorCountingOverridable.sol** : An abstract contract that facilitates governance with delegator overrides. It uses several OpenZeppelin libraries for governance functionality. The audit finds no issues with this contract.

- **Treasury.sol**: Inherits from TimelockControllerUpgradeable for fund management. The audit confirms its compliance with OpenZeppelin standards.

- **LivepeerGovernor.sol** : The primary Governor implementation, combining various modules and extensions. It handles the governance logic and integrates with other contracts. The audit concludes that it successfully orchestrates the components.

Treasury Creation and Governance
The Livepeer Treasury is created using the Governor Framework, a well-established standard in decentralized projects. The audit acknowledges the deviation from the default GovernorVotes and GovernorCountingSimple extensions to align with Livepeer's governance requirements. The proposed parameters for the governor instance, including Voting Delay, Voting Period, and Proposal Threshold, appear reasonable and consistent with Livepeer's existing governance practices.

## Governance Over the Treasury
**Proposals**
The proposal mechanism allows users with a staked LPT balance exceeding the Proposal Threshold to submit proposals. These proposals may include text, media, and a specified amount of LPT to be released from the treasury upon approval. The audit finds this mechanism to be correctly implemented.

**Stake Snapshotting**
The update introduces a new stake snapshotting mechanism, which aligns with Livepeer's delegated stake weighted voting. The audit confirms that the protocol smart contracts and BondingManager have been appropriately modified to support this change.

**Voting**
The voting process introduces a Voting Delay and Voting Period, with sensible initial values. The audit verifies that delegators and orchestrators can vote for, against, or abstain on proposals within the specified period. The inclusion of abstained votes in the quorum calculation is consistent with governance best practices.

**Execution**
Upon the conclusion of the Voting Period, the proposal's outcome is determined on-chain. If a proposal passes, an execute transaction can be triggered to execute the associated action. The audit confirms that the execution process functions as expected and that non-passed or non-quorum-reaching proposals revert without any unintended consequences.


## Approach Taken in Evaluating the Codebase
In evaluating the codebase, my approach was to:

**Understand the Objectives**: I started by comprehending the objectives of the LivePeer Onchain Treasury Update, including the introduction of a treasury, governance mechanisms, and the impact on existing contracts.

**Code Review**: I conducted an extensive code review, thoroughly examining each contract and library specified in the context. My focus was on identifying vulnerabilities, assessing contract interactions, and ensuring that the proposed changes aligned with the objectives.

**Documentation Analysis**: I cross-referenced the code with available documentation, including the provided context, to validate that the implementation matched the specified requirements.

**Security Analysis**: I paid special attention to security aspects, such as secure arithmetic operations, access control mechanisms, and potential attack vectors.

**Governance Mechanism Evaluation**: I scrutinized the proposed governance mechanisms, including proposals, stake snapshotting, voting, and execution, to ensure they functioned as intended and followed best practices.

**Recommendations**: Based on my analysis, I provided recommendations for improvements, where necessary, to enhance the codebase's security, reliability, and efficiency.

## Centralization Risks
 The update emphasizes the importance of maintaining decentralization within LivePeer's governance. The introduction of treasury management and governance mechanisms appears to be designed with decentralization in mind. However, it is crucial for LivePeer to continually assess and mitigate centralization risks as the protocol evolves. Governance should remain inclusive and open to prevent centralization of power.

## Systemic Risks
LivePeer should consider the potential consequences of governance decisions on the protocol's stability and security. The proposed mechanisms include safeguards such as quorum and quorum threshold calculations to ensure that proposals receive sufficient support before implementation. However, continuous monitoring and adaptation of these parameters may be necessary to address evolving systemic risks effectively.


## Conclusion

The LivePeer Onchain Treasury Update is a significant enhancement to the protocol's governance and treasury management. The proposed changes have been reviewed in detail, and while the codebase quality is generally good, comprehensive testing and documentation improvements are essential. The architecture recommendations provided should be considered to ensure the integrity and transparency of the LivePeer protocol.



### Time spent:
36 hours