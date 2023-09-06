# [A-01] Any comments for the judge to contextualize your findings

My primary objective is to enhance the gas efficiency and cost-effectiveness of the protocol. Throughout my analysis, I have diligently pursued opportunities to optimize gas and have prepared a comprehensive report comprising `16` detailed gas optimization with forge test and refactoring recommendations tailored to the specific needs of the protocol.

The optimizations I have identified aim to improve gas efficiency, reduce storage reads, and overall make the smart contracts more cost-effective to deploy and interact with on the blockchain

### Gas Optimization Overview

- Using Storage for Structs/Arrays: Storing data in storage variables is more efficient than using memory, especially for structs and arrays.

- Use Assembly for Address Storage: Optimize address storage operations using assembly.

- Inline Modifiers: Inline modifiers used only once to save gas.

- Avoid Bool Storage: Avoid using bools for storage as they can be more gas-expensive.

- Avoid Calculating Constants: Constants should be calculated at compile-time to save gas.

- Use Ternary Operators: Ternary operators can be more efficient than if-else statements for certain conditions.

- Short-Circuit Operations: Use short-circuit mode for Solidity operations to save gas.

- Pre-increment/Pre-decrement: Use pre-increment/pre-decrement instead of +1/-1.

- Use Assembly for Math: Use assembly for mathematical operations for gas optimization.

- Use Mappings: Consider using mappings instead of arrays for improved gas efficiency.

- Use Assembly for Event Emission: Emit events efficiently using assembly.

- Use Modifiers: Use modifiers instead of functions when applicable to save gas.

- Remove Unused Internal Functions: Remove internal functions not called by the contract to save gas.

- Use uint256(1)/uint256(2): Use this approach instead of true/false to save gas for changes.

- Use 'delete' Statement: Use 'delete' statement to clear data structures efficiently.

- Assembly for Back-to-Back Calls: Use assembly for efficient back-to-back external function calls and reuse function signatures and parameters.

These findings provide guidance for optimizing gas usage.

# [A-02] Approach taken in evaluating the codebase

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contracts Overview**:

- **BondingManager.sol**: This contract manages the protocol bonds (staking) and rewards. It has been upgraded to include support for state checkpointing and treasury rewards minting. It plays a critical role in the protocol's operation.

  - Purpose: Manages protocol bonds (staking) and rewards.
  - Key Features: Upgraded to support state checkpointing and treasury rewards minting.

- **BondingVotes.sol**: This contract stores checkpoints of the BondingManager state to provide an implementation of the IERC5805Upgradeable (IVotes) interface for the OZ Governor.

  - Purpose: Stores checkpoints of the BondingManager state to implement the IERC5805Upgradeable (IVotes) interface for the OZ Governor.
  - Key Features: Provides an interface for voting and governance functions.

- **EarningsPoolLIP36.sol**: This library provides helper functions for calculating staking rewards using the LIP-36 cumulative earnings algorithm.

  - Purpose: A library providing helper functions for calculating staking rewards using the LIP-36 cumulative earnings algorithm.
  - Key Features: Enhances the accuracy of reward calculations.

- **SortedArrays.sol**: This library provides helpers for maintaining and looking up sorted arrays. It builds on top of Arrays.findUpperBound to provide an equivalent findLowerBound for checkpoint lookup.

  - Purpose: A library providing helper functions for maintaining and looking up sorted arrays.
  - Key Features: Offers functionality for sorted array operations, including checkpoint lookup

- **GovernorCountingOverridable.sol**: This is an abstract contract that implements an OZ Governor counting module with support for delegators to override their delegates' vote on a proposal.

  - Purpose: An abstract contract that implements an OZ Governor counting module.
  - Key Features: Supports delegators overriding their delegates' votes on proposals, enhancing governance flexibility.

- **Treasury.sol**: This contract inherits from TimelockControllerUpgradeable to make it initializable. It is used by the governor to hold funds and execute proposals.

  - Purpose: Inherits from TimelockControllerUpgradeable to make it initializable; used by the governor to hold funds and execute proposals.
  - Key Features: Manages funds and proposal execution.

- **LivepeerGovernor.sol**: This contract is the actual Governor implementation, combining OZ built-in and custom modules and extensions into a concrete contract. It primarily handles the plumbing to put the pieces together.

  - Purpose: The actual Governor implementation that combines OZ built-in and custom modules and extensions into a concrete contract.
  - Key Features: Orchestrates governance operations, combining various modules.

2.  **Documentation Review**:I did not find any specific documentation for this protocol, However I watched this [video](https://lvpr.tv/?v=9ae9krf8q296csr8), which is good introduction to the protocol and the audit contest.

3.  **Graphical Analysis**: Solidity-metrics is a popular tool used to analyze and visualize Solidity code. It provides various metrics and visualizations to gain insights into the codebase's complexity and maintainability.
    [solidity-metrics](https://github.com/ConsenSys/solidity-metrics)

4.  **Utilizing Static Analysis Tools**: In this phase of the analysis, Slither has already been executed as part of the static analysis process I only see the result of it [slither-Output](https://github.com/code-423n4/2023-08-livepeer/blob/main/slither.txt).

5.  **Test Coverage Evaluation**: During this phase, I play with the tests, initially encountering challenges with Hardhat when attempting to run them first. However, after resolving the issues, I found the well-written tests to be quite interesting.

6.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode, Despite the complexity of this codebase, I didn't come across any noteworthy insights during the first two modes. Consequently, I shifted my focus to the Blackboxing mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

    - **Blackboxing Mode**: Particularly useful for complex codebases. Grasp the protocol's expected behavior through tests, and experiment with them to better understand and navigate the intricacies of the code.

# [A-03] Codebase analysis

1. **BondingManager**: Is responsible for managing bonding, transcoder operations, and rewards/fee accounting within the protocol.

   Below is a brief analysis of the key components and functions in this contract:

   - **Imports**: The contract imports several other contracts and libraries, including ManagerProxyTarget.sol, IBondingManager.sol, various libraries for mathematical calculations (SafeMath.sol, PreciseMathUtils.sol, MathUtils.sol), and other Livepeer-specific contracts related to tokens, rounds, and snapshots.

   - **Contract Constants and Variables**: The contract defines various constants and state variables, including MAX_FUTURE_ROUND, unbondingPeriod, and several data structures to manage transcoders, delegators, and earnings pools.It uses libraries like SortedDoublyLL and EarningsPoolLIP36 for managing sorted lists and earnings calculations.

   - **Structs**: The contract defines two main structs, Transcoder and Delegator, which represent the state of transcoders and delegators, respectively.
     It also defines a struct UnbondingLock for tracking unbonding details.

   - **Enums**: There are two enums: TranscoderStatus and DelegatorStatus, representing the states that a transcoder or delegator can be in.

   - **Mappings**:The contract uses mappings to keep track of transcoders and delegators based on their Ethereum addresses.
     These mappings store information such as bonded amounts, rewards, and other relevant data.

   - **Modifiers**: The contract defines various modifiers to restrict access to certain functions based on roles (e.g., onlyTicketBroker, onlyRoundsManager, onlyVerifier) and state conditions.

   - **functions**:

     1. setUnbondingPeriod(uint64 \_unbondingPeriod): This function allows the owner of the Controller to set the unbonding period, which determines how long it takes for a stake to become available for withdrawal after unbonding.

     2. setTreasuryRewardCutRate(uint256 \_cutRate): This function sets the treasury reward cut rate. It determines the percentage of newly minted rewards that should be routed to the treasury.

     3. setTreasuryBalanceCeiling(uint256 \_ceiling): This function sets the balance ceiling for the treasury reward contributions. It specifies the maximum balance at which contributions to the treasury should halt.

     4. setNumActiveTranscoders(uint256 \_numActiveTranscoders): This function sets the maximum number of active transcoders.

     5. transcoder(uint256 \_rewardCut, uint256 \_feeShare): A function to set commission rates as a transcoder and, if the caller is not in the transcoder pool, tries to add the caller as a transcoder.

     6. bond(uint256 \_amount, address \_to): Allows a user to delegate stake towards a specific transcoder.

     7. unbond(uint256 \_amount): Allows a user to unbond a certain amount of their stake.

     8. rebond(uint256 \_unbondingLockId): Rebonds tokens for an unbonding lock.

     9. rebondFromUnbonded(address \_to, uint256 \_unbondingLockId): Rebonds tokens for an unbonding lock when the delegator is in the Unbonded status.

     10. checkpointBondingState(address \_account): Checkpoints the bonding state for a given account.

     11. withdrawStake(uint256 \_unbondingLockId): Withdraws tokens for an unbonding lock that has existed through an unbonding period.

     12. withdrawFees(address payable \_recipient, uint256 \_amount): Withdraws fees to the caller.

     13. reward(): Mints token rewards for an active transcoder and its delegators.

     14. updateTranscoderWithFees(address \_transcoder, uint256 \_fees, uint256 \_round): Updates a transcoder's fee pool with fees. Only callable by the TicketBroker.

     15. slashTranscoder(...): Slashes a transcoder. Only callable by the Verifier. Slashing involves penalizing a transcoder by slashing a percentage of their bond.

     16. claimEarnings(uint256 \_endRound): Claims token pools shares for a delegator from their lastClaimRound through the specified end round.

     17. setCurrentRoundTotalActiveStake(): Called during round initialization to set the total active stake for the round.

     18. transcoderWithHint(...): Sets commission rates as a transcoder and adds the caller to the transcoder pool if they are not already in it. It allows the caller to provide hints for their position in the pool.

     19. bondForWithHint(...): Delegates stake on behalf of another address towards a specific address and updates the transcoder pool using optional list hints if needed.

     20. bondWithHint(...): This function allows a user to delegate their stake towards a specific transcoder and provides optional list hints for updating the transcoder pool. It is a part of the bonding mechanism.

     21. transferBond(...): This function transfers ownership of a bond to a new delegator and updates the transcoder pool. It's used when a user wants to transfer their bonded stake to another address.

     22. unbondWithHint(...): This function unbonds a certain amount of a delegator's stake and updates the transcoder pool with optional list hints. It allows users to initiate the unbonding process.

     23. rebondWithHint(...): This function rebonds tokens for an unbonding lock to a delegator's current delegate and updates the transcoder pool using optional list hints.

     24. rebondFromUnbondedWithHint(...): This function rebonds tokens for an unbonding lock to a delegate while a delegator is in the Unbonded status and updates the transcoder pool using optional list hints.

     25. rewardWithHint(...): This function mints token rewards for an active transcoder and its delegators, updating the transcoder pool with optional list hints.

     26. pendingStake(...): Returns the pending bonded stake for a delegator from their lastClaimRound through a specified end round.

     27. pendingFees(...): Returns the pending fees for a delegator from their lastClaimRound through a specified end round.

     28. transcoderTotalStake(...): Returns the total bonded stake for a transcoder.

     29. transcoderStatus(...): Computes and returns the transcoder's status (Registered or Not Registered).

     30. delegatorStatus(...): Computes and returns the delegator's status (Bonded, Unbonded, or Pending).

     31. getTranscoder(...): Returns information about a transcoder, including their last reward round, reward cut, fee share, and more.

     32. getTranscoderEarningsPoolForRound(...): Returns information about a transcoder's earnings pool for a specific round.

     33. getDelegator(...): Returns information about a delegator, including their bonded amount, fees, delegate address, and more.

     34. getDelegatorUnbondingLock(...): Returns information about a delegator's unbonding lock, including the locked-up amount and the withdraw round.

     35. getTranscoderPoolMaxSize(): Returns the maximum size of the transcoder pool.

     36. getTranscoderPoolSize(): Returns the current size of the transcoder pool.

     37. getFirstTranscoderInPool(): Returns the address of the transcoder with the highest stake in the transcoder pool.

     38. getNextTranscoderInPool(...): Returns the address of the next transcoder in the pool, given a specific transcoder's address.

     39. getTotalBonded(): Returns the total active stake for the current round.

     40. isActiveTranscoder(...): Checks if a transcoder is active for the current round.

     41. isRegisteredTranscoder(...): Checks if a transcoder is registered (self-bonded).

     42. isValidUnbondingLock(...): Checks if an unbonding lock for a delegator is valid (has a non-zero withdraw round).

     43. \_setTreasuryRewardCutRate(uint256 \_cutRate) internal: This internal function sets the treasury reward cut rate after validating the input. It updates the nextRoundTreasuryRewardCutRate and emits a parameter update event.

     44. cumulativeFactorsPool(Transcoder storage \_transcoder, uint256 \_round) internal view: This internal view function retrieves cumulative reward and fee factors for a specified round from a given transcoder's earnings pool data and returns it in a struct.

     45. latestCumulativeFactorsPool(Transcoder storage \_transcoder, uint256 \_round) internal view: This internal view function retrieves the latest cumulative reward and fee factors for a specified round from a given transcoder's earnings pool data. It may also use cumulative factors from the last reward or fee round if necessary.

     46. delegatorCumulativeStakeAndFees(Transcoder storage \_transcoder, uint256 \_startRound, uint256 \_endRound, uint256 \_stake, uint256 \_fees) internal view: This internal view function calculates a delegator's cumulative stake and fees based on the LIP-36 earnings claiming algorithm. It takes the start and end rounds, initial stake, and initial fees as inputs and returns the cumulative stake and fees.

     47. pendingStakeAndFees(address \_delegator, uint256 \_endRound) internal view: This internal view function calculates a delegator's pending stake and fees based on the last claim round and the specified end round.

     48. increaseTotalStake(address \_delegate, uint256 \_amount, address \_newPosPrev, address \_newPosNext) internal: This internal function increases the total stake for a delegate and updates its last active stake update round. It also handles the potential need to reorder the transcoder pool.

     49. decreaseTotalStake(address \_delegate, uint256 \_amount, address \_newPosPrev, address \_newPosNext) internal: This internal function decreases the total stake for a delegate and updates its last active stake update round. It also handles the potential need to reorder the transcoder pool.

     50. tryToJoinActiveSet(address \_transcoder, uint256 \_totalStake, uint256 \_activationRound, address \_newPosPrev, address \_newPosNext) internal: This internal function tries to add a transcoder to the active transcoder pool, evicting the one with the lowest stake if the pool is full.

     51. resignTranscoder(address \_transcoder) internal: This internal function removes a transcoder from the pool and deactivates it.

     52. updateTranscoderWithRewards(address \_transcoder, uint256 \_rewards, uint256 \_round, address \_newPosPrev, address \_newPosNext) internal: This internal function updates a transcoder with rewards and also updates the transcoder pool. It calculates and allocates rewards to both the transcoder and delegators.

     53. updateDelegatorWithEarnings(address \_delegator, uint256 \_endRound, uint256 \_lastClaimRound) internal: This internal function updates a delegator's state and delegate's state with earnings, allowing them to claim earnings.

     54. \_checkpointBondingState(address \_owner, Delegator storage \_delegator, Transcoder storage \_transcoder) internal: This internal function checkpoints the bonding state of a delegator, used for historical voting power calculations.

2. **BondingVotes**: The primary purpose of this contract is to manage voting power and stakes. It checkpoints the bonding and total stake information for accounts at specific rounds, allows querying of historical voting power and stakes, and emits events to notify other contracts of state changes.

   - **Import Statements**: This contract imports several libraries and other contracts, including OpenZeppelin's utility contracts and custom libraries.

   - **Contract Inheritance**: BondingVotes inherits from ManagerProxyTarget and implements the IBondingVotes interface.

   - **Library Usage**: It uses various utility libraries for handling arrays, mathematical operations, and custom libraries for managing earnings pools and sorted arrays.

   - **State Variables**: The contract defines several state variables to store bonding and total stake information for different accounts and rounds.

   - **Modifiers**: It defines two custom modifiers (onlyBondingManager and onlyPastRounds) to restrict access to certain functions.

   - **Constructor**: The constructor sets the Controller address when the contract is deployed.

   - **functions**:

     1. name(): This function returns the name of the token, which is "Livepeer Voting Power."

     2. symbol(): This function returns the symbol of the token, which is "vLPT."

     3. decimals(): This function returns the number of decimal places for the token, which is 18.

     4. clock(): This function returns the current round as a uint48 value. The round is related to the checkpointing mechanism implemented in the contract.

     5. CLOCK_MODE(): This function returns a machine-readable description of the clock mode, which is "mode=livepeer_round."

     6. getVotes(address \_account): This function returns the current amount of votes that a specified account has. It calculates the votes based on the account's bonding state at the next round.

     7. getPastVotes(address \_account, uint256 \_round): This function returns the amount of votes that an account had at a specific moment in the past, specified by \_round. It also calculates votes based on the bonding state at the next round.

     8. totalSupply(): This function returns the current total supply of votes available. It calculates the total supply based on the total active stake at the next round.

     9. getPastTotalSupply(uint256 \_round): This function returns the total supply of votes available at a specific round in the past, specified by \_round. It calculates the supply based on the total active stake at the next round.

     10. delegates(address \_account): This function returns the delegate that a given account has chosen. The delegate can be a transcoder address in the case of delegators or the account's own address in the case of transcoders (self-delegated).

     11. delegatedAt(address \_account, uint256 \_round): This function returns the delegate that an account had chosen in a specific round in the past, specified by \_round.

     12. delegate(address): This function reverts any attempt to delegate through BondingVotes.

     13. delegateBySig(address, uint256, uint256, uint8, bytes32, bytes32): This function reverts any attempt to delegate by signature through BondingVotes.

     14. checkpointBondingState(...): This function is called by the BondingManager when the bonding state of an account changes. It stores checkpointed bonding state information.

     15. checkpointTotalActiveStake(...): This function is called by the BondingManager when the total active stake changes. It stores checkpointed total active stake information.

     16. hasCheckpoint(address \_account): This function checks if an account already has any checkpoint.

     17. getTotalActiveStakeAt(uint256 \_round): This function gets the checkpointed total active stake at a given round.

     18. getBondingStateAt(address \_account, uint256 \_round): This function gets the checkpointed bonding state of an account at a given round.

     19. delegatorCumulativeStakeAt(...): This function calculates the cumulative stake of a delegator at any given round based on checkpointed data.

     20. getLastTranscoderRewardsEarningsPool(...): This function returns the last initialized earning pool for a transcoder at a given round.

     21. getTranscoderEarningsPoolForRound(...): This function gets the earnings pool data for a transcoder at a specific round.

     22. bondingManager(): This internal function returns the BondingManager interface.

     23. roundsManager(): This internal function returns the RoundsManager interface.

     24. \_onlyBondingManager(): This internal function ensures that the sender is the BondingManager.

3. **EarningsPoolLIP36**: This library provides functionality related to updating cumulative factors for earnings pools and calculating cumulative stake and fees for delegators

   - **updateCumulativeFeeFactor**(...): This function updates the cumulative fee factor stored in an earnings pool with new fees. It calculates the new cumulative fee factor based on the previous cumulative reward factor, the fees, and the total stake in the earnings pool. It handles the case where the cumulative fee factor is initialized.

   - **updateCumulativeRewardFactor**(...): This function updates the cumulative reward factor stored in an earnings pool with new rewards. It calculates the new cumulative reward factor based on the previous cumulative reward factor, the rewards, and the total stake in the earnings pool.

   - **delegatorCumulativeStakeAndFees**(...): This function calculates a delegator's cumulative stake and fees using the LIP-36 earnings claiming algorithm. It takes the earning pools from the start and end rounds, the delegator's initial stake, and the initial fees as inputs. It calculates cumulative fees and cumulative stake based on the changes in cumulative factors between the start and end rounds.

4. **GovernorCountingOverridable**: This is an abstract contract that extends the functionality of the OpenZeppelin Governor contract with support for a specific voting mechanism that allows delegators to override their delegated transcoder's votes.

   - **Initializable and Governor Upgradeable**: This contract uses the OpenZeppelin Initializable and GovernorUpgradeable contracts, indicating that it's designed to work with upgradeable contracts. This means that it can be upgraded without losing state data.

   - **Import Statements**: This contract imports several external contracts and interfaces from OpenZeppelin and other custom contracts. These include contracts related to governance, bonding, and voting.

   - **VoteType Enumeration**: It defines an enumeration called VoteType, which represents different types of votes: "Against," "For," and "Abstain."

   - **Structs**:

   **ProposalVoterState**: This struct tracks the voting state of specific voters in a single proposal. It includes whether the voter has voted, their vote type, and vote deductions (for delegators who might vote before their transcoder).

   **ProposalTally**: This struct tracks the tallying state for a proposal, including the number of votes "Against," "For," and "Abstain," as well as the voter states.

   - **Mappings**: \_proposalTallies: This private mapping associates proposal IDs with their corresponding vote tallies.

   - **Functions**:

     1. COUNTING_MODE Function: This function returns a string representing the counting mode for governance. It specifies the voting rules as "support=bravo&quorum=for,abstain,against," indicating that the governance system follows Bravo's voting support logic and requires quorums for "for," "abstain," and "against" votes.

     2. hasVoted Function: This public view function checks whether a specific account has already voted on a particular proposal. It does so by looking up the proposal's ID and the voter's address in the internal \_proposalTallies mapping.

     3. proposalVotes Function: This public view function retrieves the vote counts for a specific proposal. It returns the number of votes "against," "for," and "abstain" for the given proposal by querying the internal \_proposalTallies mapping.

     4. \_quorumReached Function: This internal view function checks whether the quorum requirement for a proposal has been met. It calculates the total number of votes (including "against," "for," and "abstain") for the proposal and compares it to the quorum specified in the proposal's snapshot.

     5. \_voteSucceeded Function: This internal view function checks whether a vote on a proposal has succeeded. In this module, a successful vote is defined as having "for" votes that are at least the specified quota percentage of the total opinionated votes (excluding "abstain" votes).

     6. \_countVote Function: This internal function is responsible for counting a user's vote on a proposal. It takes the proposal ID, voter's address, vote type (as an integer), vote weight, and additional parameters (unused) as arguments. It first checks the validity of the vote type and then updates the vote tally for the proposal based on the vote type ("Against," "For," or "Abstain"). It also handles vote overrides when a delegator overrides their transcoder's vote.

     7. \_handleVoteOverrides Function: This internal function handles vote overrides made by delegators to their delegated transcoder's votes. It calculates the effective voting power of each vote by considering the transcoder-delegator relationship. If the voter is a transcoder, it deducts any previous delegator votes for that transcoder. If the voter is a delegator, it adds a deduction to the delegated transcoder and updates the current vote totals accordingly.

     8. votes Function: This is an abstract function that must be implemented in inheriting contracts. It's expected to provide an interface (IVotes) to access voting power and delegation information.

     9. Storage Gap: The contract includes a storage gap of 48 uint256 variables, which is reserved for potential future upgrades without affecting existing storage.

5. **LivepeerGovernor**: The LivepeerGovernor contract is a core component of the Livepeer governance system, leveraging multiple OpenZeppelin extensions for governance, voting, and timelock control. It allows for flexible parameterization and updating of contract addresses for future-proofing the system.

   - **Contract Inheritance and Imports**: The contract imports various OpenZeppelin contracts, such as Initializable.sol, GovernorUpgradeable.sol, GovernorVotesUpgradeable.sol, and others, to build upon their functionality for governance.

   - **Constructor**: The constructor initializes the contract with a reference to the controller contract. It sets up the initial state without executing the actual initialization. Initialization must be called explicitly through the initialize function.

   - **Initialization Function**: The initialize function sets up the governance contract's parameters. It defines values such as the initial voting delay, voting period, proposal threshold, quorum, and quota. These parameters are crucial for the governance process.

   - **quorumDenominator Function**: This function returns the quorum denominator used in governance calculations.
     It sets the quorum denominator to MathUtils.PERC_DIVISOR, indicating a fixed denominator for quorum calculations, likely representing a percentage.

   - **votes Function**: This function specifies the contract responsible for holding votes.
     It returns the bondingVotes contract, which is an internal function used to access the voting contract.

   - **bumpGovernorVotesTokenAddress Function**: This function allows updating the address of the BondingVotes contract used for voting.
     It sets the token property of the governance contract to the result of the votes function.
     This function is callable by anyone and is designed to future-proof the contract in case the BondingVotes contract's address changes in the controller.

   - **bondingVotes Function**: This internal function retrieves the address of the BondingVotes contract from the controller and returns it as an IVotes interface.
     It uses the controller.getContract function to fetch the contract address based on a keccak256 hash of the contract's name.

   - **treasury Function**:This internal function retrieves the address of the Treasury contract from the controller and returns it as a Treasury contract instance.
     It uses the controller.getContract function similarly to the bondingVotes function.

# [A-04] Centralization risks

1. **BondingManager**:

   - **Control over Parameters**: The contract includes several functions that allow the owner (Controller owner) to modify critical parameters such as the unbonding period, treasury reward cut rate, and the number of active transcoders. Centralized control over these parameters can lead to a single entity having significant influence over the network's behavior

   - **Transcoder Management**: Transcoders play a crucial role in the Livepeer protocol. The transcoder and transcoderWithHint functions allow transcoders to set their reward cut and fee share percentages. While this allows for flexibility, it can also lead to centralization if a few large transcoders dominate the network

   - **Transcoder Resignation**: The resignTranscoder function allows a transcoder to voluntarily resign from the pool. This functionality could be exploited if a dominant transcoder decides to resign to manipulate the transcoder pool in their favor, potentially leading to centralization

   - **Transcoder Pool Management**: The code maintains a pool of transcoders, which determines who can participate in the network as active transcoders. The tryToJoinActiveSet function allows transcoders to join the active set, but there doesn't seem to be a strict mechanism for ensuring decentralization. Transcoders with higher stakes have an advantage, potentially leading to centralization.

   - **Auto Claiming**: The use of the autoClaimEarnings modifier and autoClaimEarnings(msg.sender) in various functions implies that earnings are automatically claimed for users. Automatic claiming could be seen as a centralizing feature, as it may not give users control over when they claim their earnings.

   - **Manual Delegation Changes**: The code allows users to manually change their delegate addresses, which may lead to centralization if users frequently change delegates to follow a dominant transcoder.

2. **BondingVotes**:

   - **Exclusive Control by BondingManager**: The onlyBondingManager modifier restricts certain critical functions to only be callable by the BondingManager. If the BondingManager is controlled by a single entity or a centralized authority, this centralizes control over key aspects of the contract, such as checkpointing bonding states and total active stake.

   - **Delegate Logic and Influence**: The delegates and delegatedAt functions return the delegate address for an account. If a small group of accounts or entities has a significant amount of tokens delegated to them, they may have substantial influence over the network. This could centralize decision-making power.

3. **LivepeerGovernor**

   - **Quorum Setting**: The quorum fraction is set to a fixed value in the quorumDenominator function. If this value is not adjustable through governance and remains fixed, it may not accurately reflect the evolving consensus of the community.

   - **Executor Address**: The \_executor function returns an address that may be controlled by a central authority. If this address can be modified by a centralized party, it could centralize control over proposal execution.

# [A-05] Systemic risks

- **Staking Rewards and Inflation**: The mention of inflationary LPT rewards being shared with transcoders and their delegators underscores the importance of a balanced and sustainable reward system. Systemic risks may arise if the reward distribution mechanism does not adequately align incentives, potentially leading to centralization or economic imbalances within the network.

- **Delegate Bonding**: The bondForWithHint function allows for delegate bonding and changing delegates. Managing delegate changes correctly is crucial, as incorrect delegation could lead to stake being misallocated, which can impact rewards distribution.

- **Unbonding Period**: The unbondingPeriod parameter defines the time between unbonding and possible withdrawal. If not set correctly, it could lead to unintended consequences for users who want to unbond their stakes.

- **Hint-Based List Management**: The code relies on hints for managing lists of transcoders, which are used for various operations. If the hints are not updated correctly or are manipulated, it could lead to transcoder list corruption and potential manipulation of the protocol.

- **Linear Search for Hints**: Linear searches are performed based on the provided hints. Depending on the size of the list, this can be inefficient and costly in terms of gas usage. Gas costs should be carefully considered.

- **EarningsPoolLIP36.sol**: The code involves complex logic for updating cumulative factors and calculating cumulative stake and fees. Complex logic increases the risk of errors and makes it harder to identify potential vulnerabilities.

  1. Cumulative Factor Initialization: The code initializes cumulative reward and fee factors to PreciseMathUtils.percPoints(1, 1) when they are zero. Ensure that this initialization logic is appropriate for your use case and that it won't lead to unexpected behavior.


### Time spent:
15 hours