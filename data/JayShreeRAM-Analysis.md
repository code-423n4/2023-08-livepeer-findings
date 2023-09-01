## Livepeer Onchain Advanced Analysis Report

## 1  .  Introduction ##

 The Livepeer onchain treasury update introduces a governance mechanism for managing funds and executing proposals within the Livepeer protocol. This mechanism utilizes several new contracts and updates existing ones to facilitate on-chain governance. Below, I reviewed the key components, terms, parameters, and contracts involved in this onchain update. 



## 2 .  Unique Patterns ##

**1 . Delegator Override**

 One of the unique patterns in this governance system is the ability for delegators to override their transcoder's votes. This means that even if a delegator has delegated their tokens to a transcoder, they can cast their own vote on a proposal, potentially differing from their transcoder's vote. This allows for more fine-grained control over voting preferences.

**2 . Customized Vote Counting**

 The LivepeerGovernor contract includes a custom implementation for vote counting, known as GovernorCountingOverridable. This custom module enables the counting of votes where both transcoders and delegators can participate, and it considers delegator overrides. This differs from typical on-chain governance systems that may not allow delegators to vote separately from their transcoders.

**3 . ERC-5805 Compliance**

 The governance system implements aspects of the ERC-5805 standard to handle voting power and delegation. While ERC-5805 is not widely adopted, it offers a unique approach to on-chain governance and delegation that differs from more common governance standards.

## 3 . Existing Patterns ##

**OpenZeppelin Governance Primitives**

 The governance system leverages the Governance primitives provided by OpenZeppelin, which includes modules for proposal creation, voting, and execution. This is a common pattern in many on-chain governance systems to ensure security and modularity.

**Quorum and Proposal Thresholds**

 The system implements common patterns such as quorum requirements and proposal thresholds. Quorum ensures a minimum level of participation in voting, while the proposal threshold sets the minimum amount of voting power required to create a proposal. These patterns help maintain the integrity of the governance process.

**Timelock for Proposal Execution**

 The use of a timelock mechanism, inherited from TimelockControllerUpgradeable, is a common pattern in governance systems. It enforces a delay between proposal approval and execution, providing a safety net for the community to react if necessary.

**Checkpointing**

 The system uses checkpointing to record and track the state of various parameters, including bonding state and total active stake. Checkpointing is a common pattern to ensure that the historical state can be referenced accurately for voting and tallying.

**Decentralized Governance**

 The governance system follows the pattern of decentralized governance, where decisions are made collectively by token holders. This is a fundamental pattern in on-chain governance systems.


## 4 .  Mechanism Review ##
## 4.1 Bonding Manager 
 **1 . Checkpointing Total Active Stake**

 -  To checkpoint the total active stake of every round, the ROUNDS_MANAGER.initializeRound() can be used to indirectly checkpoint the total active stake for the protocol on that round.
 -  After setCurrentRoundTotalActiveStake is called in round r, the expectation is that:

            1 . BondingVotes.totalSupply() returns exactly BONDING_MANAGER.currentRoundTotalActiveStake() during the same round.
            2 . BondingVotes.getPastTotalSupply(r) will immutably return the same value above in the future.
 **2 . Checkpointing Bonding State**

   -  The main changes to BondingManager were internal, to make sure we checkpoint all the bonding state every time any change is made to an account state.

   -   The optimal gas implementation of this would also make sure the BondingManager never checkpoints the same address multiple times on a single function call.

   -  In practical terms, it can be defined that after any write function called on the BONDING_MANAGER (including the checkpointBondingState below), the checkpointed state of all involved accounts should follow that mutation consistently.

  **3 . Interface additions**


        contract BondingManager {
           function checkpointBondingState(address _account) external;
        }


  - checkpointBondingState: Explicitly checkpoints an account bonding state in the BondingVotes contract. This can be used to fix the checkpointed state of any account that may have diverged unexpectedly.

## 4.2 BondingVotes

The BondingVotes contract is a key component of the Livepeer protocol, designed to manage and track voting power within the system. It leverages checkpointing mechanisms to capture and record the state of various stakeholders, including delegators and transcoders, in the Livepeer network. This report will provide an in-depth analysis of the contract's functions, their purposes, and their expected behaviors.
   
 **1 . Checkpointing State**
  **- 1.1 checkpointBondingState**

 - It must be callable only by the BONDING_MANAGER.
 - It requires the _startRound parameter to match the next round to ensure consistency.
 -It emits events such as DelegateChanged, DelegateVotesChanged, and DelegatorBondedAmountChanged when appropriate, signaling changes in delegate addresses, delegated amounts, and bonded amounts.

 **- 1.2 checkpointTotalActiveStake**

- It should be called for every round, as the total active stake continually changes.
- It is called from BONDING_MANAGER.setCurrentRoundTotalActiveStake() to ensure accuracy.
 - It requires the _round parameter to be the current round.

 **2. State Lookup**
The BondingVotes contract provides functions for looking up the checkpointed state to ensure consistency and integrity of data.

   **- 2.1 getTotalActiveStakeAt**

This function returns the total active stake at the start of a specified round. It considers various scenarios, including the absence of checkpoints, existence of checkpoints, and checkpoint values before and after the searched round.

  **- 2.2 hasCheckpoint**

This utility function checks whether a specific account has any checkpoints already registered, helping initialize account checkpoints when necessary.

 **- 2.3 getBondingStateAt**

This function retrieves the active stake and delegate of an account in a specified round, calculated using the checkpoint with the highest startRound lower or equal to the specified round. It behaves differently for transcoders and delegators, ensuring accurate data retrieval.

**3. ERC-20 Metadata**

The BondingVotes contract implements optional ERC-20 metadata methods to improve interoperability with existing tools. These methods return specific metadata about the token.

          name: Returns "Livepeer Voting Power."
          symbol: Returns "vLPT."
          decimals: Returns 18.

**4 . ERC-5805**

The BondingVotes contract partially implements ERC-5805, which specifies how to express voting power with checkpointing and delegation mechanisms. The contract adheres to the following ERC-5805 functions:

 - clock: Returns the current round from ROUNDS_MANAGER.currentRound().
- CLOCK_MODE: Returns "mode=livepeer_round."
- getVotes(_account): Returns the amount value from getBondingStateAt(_account, clock() + 1).
- getPastVotes(_account, _round): Returns the amount value from getBondingStateAt(_account, _round + 1).
- getPastTotalSupply(_round): Extends ERC-5805 and returns getTotalActiveStakeAt(_round + 1).
- totalSupply(): Extends ERC-5805 to allow accessing the current total supply, returning getTotalActiveStakeAt(clock() + 1).
- delegates(_account): Returns delegateAddress value from getBondingStateAt(_account, clock() + 1).
- delegatedAt(_account, _round): Extends ERC-5805 to allow delegators to override transcoders' votes consistently at proposalSnapshot round, returning delegateAddress value from getBondingStateAt(_account, _round + 1).

**5. Caveats**

The BondingVotes contract has some known divergences from ERC-5805 and IERC5805Upgradeable expectations:

- It does not implement the mutation functions delegate and delegateBySig but instead reverts with specific error messages.
- It lacks support for the nonces function.
- It provides voting power to both delegators and transcoders, which is not consistent with ERC-5805 but is required for the Livepeer governance system.
- The contract partially implements events specified in ERC-5805, which may not provide a complete representation of voting power due to complex factors like delegated stake and pending rewards.


## 4.3 GovernorCountingOverridable (extension) ##

 This is an important component of a governance system that complements the BondingCheckpointsVotes module by providing vote counting logic. It is designed to be as similar as possible to the off-chain indexer logic described in LIP-16, providing a robust and flexible voting mechanism for decentralized decision-making within a blockchain ecosystem.

**Key Components and Functions:**

**1 . IVotes Interface**
 This interface defines the methods for interacting with the voting system. It includes functions for retrieving voting-related information such as name, symbol, decimals, total supply, and delegated voting power at a specific timepoint.

**2 . GovernorCountingOverridable Contract**
 This abstract contract extends the GovernorUpgradeable contract and implements the vote counting logic. It introduces several key functions and properties:

- _GovernorCountingOverridable_init(_quota): This internal function initializes the module with a provided quota, which is expressed in 6-digit decimal precision compatible with the protocol's MathUtils. The quota is crucial for calculating whether a vote succeeds or not.

- quota(): This public view function returns the configured quota value for calculating the success of a vote. The quota is set during the initialization of the Governor and is not updatable without a future upgrade.

- COUNTING_MODE(): This function returns a string representing the counting mode, which is "support=bravo&quorum=for,abstain,against" in this case.

- hasVoted(_proposalId, _account): This function checks whether a specific account has already cast a vote on a particular proposal. It indicates whether _countVote has been called for the same account and proposal.

- proposalVotes(_proposalId): This function returns the sum of votes made for, against, and abstain for a given proposal. It collects voting data from _countVote for each vote type.

- _quorumReached(_proposalId): This internal view function determines whether the sum of votes (for, against, and abstain) meets the minimum defined threshold (quorum). It retrieves the quorum from the GovernorUpgradeable contract's quorum() virtual function and considers all vote types for quorum calculation.

- _voteSucceeded(_proposalId): This internal view function determines whether a proposal has been approved based on the voting outcome. It calculates the quota percentage from the configured quota variable and considers only the "For" and "Against" votes.

- _countVote(...): This internal virtual function implements the logic for counting votes when a vote is cast on a proposal. It considers different scenarios, including when a transcoder votes first, when delegators vote first, when delegators vote after their transcoder, and when a transcoder votes after their delegators.

 The votes() function, which returns an instance of the IVotes interface, needs to be implemented by a concrete LivepeerGovernor contract, providing the contract to use for voting power.


## 4.4 Treasury ##

The primary purpose of the Treasury Timelock Controller is to act as an essential component of the governance mechanism for the Livepeer protocol. It serves as an interface to interact with the TimelockControllerUpgradeable contract, allowing the system to enforce time delays for certain administrative actions, ensuring a level of security and oversight.

Treasury roles and their responsibilities as configured in production deployments:

**1 . TIMELOCK_ADMIN_ROLE**

- Role Holder: Only the Timelock controller itself should possess this role.
- Responsibility: This role grants administrative privileges for updating roles within the Livepeer protocol.
However, these administrative actions must follow a time delay enforced by the timelock itself. This means that role updates must be proposed and executed through the timelock, adding a layer of security and oversight.

**2 . PROPOSER_ROLE**

- Role Holder: Only the LivepeerGovernor contract should have this role.
- Responsibility: The PROPOSER_ROLE allows the LivepeerGovernor to enqueue functions for execution through the timelock. These functions typically correspond to actions resulting from successful proposals. This role ensures that only the LivepeerGovernor can initiate actions that impact the protocol.

**3 . EXECUTOR_ROLE**

- Role Holder: Only the LivepeerGovernor contract should have this role.
- Responsibility: The EXECUTOR_ROLE grants the LivepeerGovernor the authority to update its internal state when a proposal is executed. While the EXECUTOR_ROLE's own execute() function can be called by anyone, the LivepeerGovernor typically handles this process, maintaining consistency in the governance process.

**4 . CANCELLER_ROLE**

- Role Holder: Also only given to the LivepeerGovernor.
- Responsibility: Although currently unused in the implementation, the CANCELLER_ROLE could potentially be assigned to the LivepeerGovernor or other agents in the future. This role would be relevant if a non-zero timelock delay is configured, allowing designated parties to cancel queued actions.


## 4.5 LivepeerGovernor ##


The LivepeerGovernor contract is a complex smart contract that brings together various elements to facilitate on-chain governance. It inherits functionality and features from several other contracts, both non-functional and functional:

**1 . Non-functional Inheritance**

ManagerProxyTarget: This contract supports contract upgradability, allowing for future upgrades consistent with the rest of the protocol contracts.
Initializable (OpenZeppelin): Provides abstractions for initializing proxied contracts.

**2 . Functional Inheritance**

- GovernorUpgradeable (OpenZeppelin): This is the core OpenZeppelin governance contract, providing fundamental governance functionality.
- GovernorSettingsUpgradeable (OpenZeppelin): Manages parameters like voting delay, voting period, and proposal threshold without requiring a contract upgrade.
- GovernorTimelockControlUpgradeable (OpenZeppelin): Integrates a TimelockController for delayed proposal execution, with the Treasury as the timelock controller.
- GovernorVotesUpgradeable (OpenZeppelin): Implements a votes module that extracts voting weight from an IERC5805Upgradeable (implemented by BondingVotes).
- GovernorVotesQuorumFractionUpgradeable (OpenZeppelin): Manages an updatable quorum parameter without requiring an upgrade.
- GovernorCountingOverridable: Provides a custom implementation for the governor counting module to allow delegators to override their delegated transcoder votes.

## 5 . Codebase Quality Analysis

**1. Modularity and Code Separation**

The codebase is well-structured and modular, with distinct contracts handling various governance functions.
Each contract has a specific purpose and encapsulates related functionality.

**2. Inheritance and Reusability:**

The codebase effectively utilizes inheritance to reuse functionality from OpenZeppelin contracts, known for their security and reliability.

**3. Documentation and Comments**

Detailed documentation is provided, explaining the purpose and functionality of each contract, parameter, and function.
Comments within the code clarify complex logic and the reasoning behind certain decisions.

**4. Initialization and Upgradability**

Initialization is handled well using the Initializable contract from OpenZeppelin.
The contract LivepeerGovernor incorporates the ManagerProxyTarget for upgradability, ensuring future updates can be made without disrupting the existing system.

**5. Parameterization and Configurability**

The codebase allows parameterization of key variables such as voting delay, voting period, threshold, and quorum.
These parameters can be updated through governance actions, providing flexibility for adjustments in the future.

**6. Security**

The codebase relies on trusted OpenZeppelin contracts for core governance functionality, which have undergone extensive security audits.
Proper access control mechanisms are in place to restrict actions to authorized roles.

**7. Error Handling**

Error handling is implemented to revert transactions when unauthorized actions are attempted.
Errors are handled gracefully with informative error messages.

**8. Consistency and Standards**

The codebase adheres to established standards such as ERC-5805 for voting power and ERC-20 for token metadata.
Naming conventions and function signatures follow common Ethereum development practices.

**9. Testing and Deployment**

Thorough testing should be conducted to ensure the correctness and security of the contracts before deployment in a live environment.

**10. Governance Processes**

- The codebase defines clear governance processes for decision-making, including voting and execution.
- A timelock mechanism is employed to enforce delays in certain actions, adding an extra layer of security.

**11. Caveats and Considerations**

- The codebase addresses and documents deviations from ERC-5805 specifications, providing explanations for these choices.

**12. Community Involvement**

- The proposed governance model encourages community participation in decision-making through voting and other actions.





## 6 .  Centralization Risks ##
LivepeerOnchain Treasury  appears to be well-structured, there are several centralization risks that should be considered:

**1 . Token Distribution**
 - The effectiveness of on-chain governance often depends on a well-distributed token supply. If a significant portion of tokens is held by a small group of addresses, they could potentially dominate the decision-making process, leading to centralization.

**2 . Delegate Voting Power**
-  The system allows both transcoders and delegators to vote. However, if a large number of delegators follow the voting decisions of a small number of transcoders, it could result in centralization of decision-making power.

**3 . Minimum Quorum**
- The minimum quorum required for a proposal to be valid may be set too low. This could lead to proposals being approved with minimal participation, potentially allowing a small group to make decisions on behalf of the entire community.

**4 . Proposal Threshold**
- The proposal threshold represents the amount of voting power required to submit a proposal. If this threshold is too high, it may deter smaller token holders from participating in the governance process, leading to centralization of proposal creation.

**5 . Governor Timelock**
- The introduction of a timelock for proposal execution can be beneficial for security. However, if the timelock is controlled by a centralized entity or has an excessively long delay, it could hinder the protocol's ability to respond to urgent issues.

**6 . Quota and Quorum Changes**
- The ability to change the quota and quorum parameters through governance proposals introduces a risk. If these parameters are altered in a way that favors a specific group, it could lead to centralization.

**7 . Checkpointing and Governance Power**
- The use of checkpointing for voting power calculation may introduce centralization risks if certain actors can manipulate checkpointing processes or have disproportionate influence over the data used for voting power.

**8 . Lack of Voting Participation**
-  If a significant portion of token holders do not actively participate in governance, decisions may be made by a relatively small group, leading to centralization of power.

**9 . Single Implementation**
- The reliance on a single implementation of the governance system may pose centralization risks. If this implementation has vulnerabilities or is controlled by a single entity, it could impact the integrity of the governance process.

**10 . Governance Proposals**
-  The governance system allows for the creation and execution of proposals. If the proposal process is not transparent or inclusive, it may lead to centralization of decision-making authority.

## 7 . Learning & insights

After conducting an audit of the Livepeer Onchain Treasury Update, I have gained valuable insights into the intricacies of this on-chain governance system and its associated smart contracts. Here's what I've learned from this audit:

**Governance Contracts**
 The Livepeer Onchain Treasury Update introduces several key contracts that form the foundation of the governance system. These contracts include BondingVotes, Treasury, and LivepeerGovernor. Each contract serves a specific role in the governance process, from managing voting power to executing proposals.

**Checkpointing State**
 The audit revealed that the BondingManager contract has undergone significant changes to implement checkpointing of total active stake and bonding state. This checkpointing process ensures that historical data is accurately recorded and maintained for voting power calculations.

**Parameters and Definitions**
 The audit provided a thorough breakdown of the parameters and definitions used within the governance system. It clarified terms such as active stake, quorum, voting delay, and proposal threshold, giving a clear understanding of how these parameters impact the decision-making process.

**ERC-20 Metadata and ERC-5805**
 I learned that the governance contracts implement optional ERC-20 metadata methods to enhance interoperability with existing tools. Additionally, the implementation follows certain aspects of the ERC-5805 standard to express voting power, although there are acknowledged deviations from the standard.

**Governor Counting and Overrides**
 One notable aspect of the audit is the custom implementation of the GovernorCountingOverridable contract, which allows delegators to override their transcoder's votes. This customization demonstrates flexibility in the governance design to accommodate various voting scenarios.

**Roles and Permissions**
The audit highlighted the importance of roles and permissions within the governance system. Roles are assigned to specific contracts, such as the TimelockController, to control access and actions related to governance proposals.

**Initialization**
 The LivepeerGovernor contract includes an initialization function with various initial parameters. This step is crucial for setting up the governance system with appropriate configurations.

**Caveats**
 The audit report acknowledges certain deviations from ERC-5805 and IERC5805Upgradeable standards and provides explanations for these variations. These deviations are important to consider when assessing the compliance of the governance system.







## 8 . comments for the judge


The architecture proposed for this update appears to be well-thought-out and aligned with established governance best practices. The use of OpenZeppelin's Governance primitives and the clear definitions and explanations of various components, terms, and parameters contribute to the solidity of the proposal.

The architecture is built on solid foundations, with contracts like BondingVotes and LivepeerGovernor extending from established OpenZeppelin contracts, ensuring reliability and security. The implementation of checkpointing for bonding state and active stake adds robustness to the system, especially when dealing with on-chain governance.



However, there are a few caveats and considerations to note:

- The  ERC-5805 and IERC5805Upgradeable  should be carefully reviewed to ensure they do not introduce unexpected complexities or vulnerabilities.

- The implementation of the timelock with an initial value of zero may raise questions about the potential need for an enforced delay in the future. It's important to assess whether this design choice aligns with the overall governance strategy.

-  It might be beneficial to include additional details on the rationale behind certain choices, especially when deviating from standard practices. This can help reviewers better understand the decision-making process.

 I recommend a thorough review of the mentioned deviations from standards to ensure they do not introduce unexpected risks.

## 9 . Conclusion

The Livepeer Onchain Treasury Upgrade exhibits a well-structured architecture, sound codebase quality, and mechanisms that align with governance principles. While no critical issues were identified in this advanced analysis, it is essential to conduct a comprehensive code review, monitor governance participation, and implement robust testing and monitoring procedures to ensure the security and effectiveness of the upgrade.

The audit provides recommendations for further improvements and risk mitigation measures. The Livepeer community should consider these recommendations while proceeding with the upgrade to enhance the governance and treasury management within the Livepeer ecosystem.

## 10. Time spent 

     12 Hours

### Time spent:
12 hours