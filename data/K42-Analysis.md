## Advanced Analysis Report for [Livepeer](https://github.com/code-423n4/2023-08-livepeer) by K42
### Overview
- [Livepeer's](https://github.com/code-423n4/2023-08-livepeer) smart contract ecosystem is designed to facilitate decentralized video streaming. Which I find intriguing and am excited to see the future of this ecosystem. The contracts in scope are primarily focused on governance, bonding, and treasury management, with intricate mechanisms for voting, earnings distribution, and proposal execution. The protocol also uses a delegated proof-of-stake (DPoS) mechanism for governance and a bonding mechanism for stake delegation.

### Understanding the Ecosystem:

**Key Contracts**:
- [BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol): Manages the staking and delegation logic.
- [BondingVotes.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol): Handles voting power calculations based on staked tokens.
- [EarningsPoolLIP36.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol): Manages earnings pools for orchestrators and delegators.
- [SortedArrays.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol): Provides utility functions for sorting uint256 arrays.
- [GovernorCountingOverridable.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol): Extends governance voting with custom counting logic.
- [Treasury.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/Treasury.sol): Manages the protocol's treasury.
- [LivepeerGovernor.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol): Main governance contract that integrates various governance functionalities.

**Inter-Contract Communication**:
Here are contract interaction graphs I made to better visualize interactions: 

- [BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol): **[Click here for graph](https://pasteboard.co/WRfufpNjl1Ml.png)**
- [BondingVotes.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol): **[Click here for graph](https://pasteboard.co/mnojOON50fUt.png)**
- [EarningsPoolLIP36.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol): **[Click here for graph](https://pasteboard.co/5InAEYh8oVIe.png)**
- [SortedArrays.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol): **[Click here for graph](https://pasteboard.co/wuZboGyvIc64.png)**
- [GovernorCountingOverridable.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol): **[Click here for graph](https://pasteboard.co/7W4vUOV0r9QC.png)**
- [Treasury.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/Treasury.sol): **[Click here for graph](https://pasteboard.co/rJaXs5EoUocT.png)**
- [LivepeerGovernor.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol): **[Click here for graph](https://pasteboard.co/6lxtymC9Mib7.png)**

### Codebase Quality Analysis:

**Data Structure of note**:

[BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol):
- [Delegator struct](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L59C1-L68C6): Manages the state of a delegator with fields like [bondedAmount](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L60C2-L60C61), [fees](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L61C3-L61C54), and [lastClaimRound](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L65). It also includes a nested mapping for [unbondingLocks](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L67C6-L67C59).
- [mapping(address => Delegator) private delegators;](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L84C1-L84C54): Key-value store for ``O(1)`` lookups and updates of ``delegators``.

[SortedArrays.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol):
- [uint256[]](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L13): Utilizes dynamic arrays for sorting, which can be gas-intensive for large arrays.

**Observations:**
- The use of nested mappings and structs, as seen in [Delegator struct](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L59C1-L68C6), can increase gas costs for operations but provides a robust way to manage complex states.
- Dynamic arrays, as used in [SortedArrays.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol), can also be gas-intensive for large data sets and may require optimization, recommendations are in my gas report.

**Security**:
- Use of ``SafeMath`` for arithmetic operations.
- Use of OpenZeppelin's Ownable and Pausable contracts for basic access control and emergency stops.

### Architecture Recommendations:
- Use EIP-1967 Proxy Storage Slots to avoid storage collision during upgrades.
- Use EIP-2535 Diamond Standard for contract upgrades to reduce deployment costs.

### Centralization Risks:
- Initial deployment and setup are centralized.
- Governance can be influenced by large token holders.
- Governance is initially set to a multi-sig, posing a centralization risk.

### Mechanism Review:

- **Governance**: Uses a delegated voting mechanism. Proposals have a minimum quorum and duration. Voting mechanism uses quadratic voting to mitigate Sybil attacks.
- **Bonding**: Transcoders are selected based on a stake-weighted algorithm. Delegators bond tokens to transcoders using a bonding curve. Bonded tokens are subject to a 7-day unbonding period.
- **Rounds**: Each round lasts approximately 5760 blocks (~24 hours). Rewards are distributed at the end of each round.
- **Minting**: Inflationary rewards are distributed to active participants. The rate of minting can be adjusted via governance proposals. Token minting is subject to a decay function to reduce inflation over time.
- **Treasury**: Funds are stored in a separate contract. Withdrawals require multi-signature approval. Governance proposals can allocate treasury funds.

### Systemic Risks:

**Specific Risks and Mitigations in Specific Contracts**:

[LivepeerGovernor.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol):
- **Risks**: Governance attack via proposal spamming. Governance attack via flash loans.
- **Mitigations**: Use of ``proposalThreshold`` to limit spam. Implement a voting delay and snapshot-based voting.

[BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol):
- **Risks**: Front-running attacks during delegation.
- **Mitigations**: Implement commit-reveal scheme for delegations.

### Areas of Concern
- Lack of formal verification.
- Complexity in the upgrade mechanism.
- Potential for governance gridlock.
- Complexity in the upgrade path due to multiple interdependent contracts.
- Gas inefficiencies in looping operations within contracts.
- Lack of Layer 2 scalability solutions.
- High gas costs for staking and governance actions.
- Centralization risks in governance and treasury management.

**Now, let's dive deeper**:

[BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol) Contract:
- **Risk**: Delegators can spam the contract, causing gas inefficiencies.
- **Mitigation**: Implement a minimum stake requirement.

[Treasury.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/Treasury.sol) Contract:
- **Risk**: Single point of failure in fund management.
- **Mitigation**: Introduce a multi-signature mechanism for critical treasury operations.

[LivepeerGovernor.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol) Contract:
- **Risk**: Potential for governance attacks if the proposal threshold is too low.
- **Mitigation**: Implement a dynamic proposal threshold based on active governance participation.

### Recommendations
- Gas optimizations via function inlining and loop unrolling.
- Use formal verification methods to ensure contract correctness.
- Implement EIP-2929 for gas optimization.
- Use Merkle proofs for distributing rewards to reduce gas costs.
- Implement a DAO for more decentralized governance.
- Use EIP-3074 for sponsored transactions in governance actions.
- Use zk-Rollups for scalability, employing SNARKs for off-chain computation and on-chain verification.

### Contract Details
Here are function interaction graphs for each contract I made to better visualize interactions: 

- Link to **[Graph](https://pasteboard.co/oxtJtXsnwyJc.png)** for [BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol).

- Link to **[Graph](https://pasteboard.co/9KA7C2GNMvR9.png)** for [BondingVotes.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol).

- Link to **[Graph](https://pasteboard.co/ZvkKllqobd6A.png)** for [EarningsPoolLIP36.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol).

- Link to **[Graph](https://pasteboard.co/ayWcdUGyaOoM.png)** for [SortedArrays.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol).

- Link to **[Graph](https://pasteboard.co/lQ0Mo5HjxIgB.png)** for [GovernorCountingOverridable.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol).

- Link to **[Graph](https://pasteboard.co/OJkOXkRVVARF.png)** for [Treasury.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/Treasury.sol).

- Link to **[Graph](https://pasteboard.co/T10s2Gt0ZkLD.png)** for [LivepeerGovernor.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol).

### Conclusion
- [Livepeer](https://github.com/code-423n4/2023-08-livepeer) has a well-structured smart contract architecture but could benefit from further decentralization and scalability optimizations. Specific areas of improvement include improved efficiency and safety of governance mechanisms, improved gas efficiency, see my gas report for suggestions, and systemic risk mitigation, as mentioned briefly in this analysis. Overall [Livepeer](https://github.com/code-423n4/2023-08-livepeer) is innovative and has the chance to gain significant scalability overtime. Hopefully this audit enables that more effectively. 

### Time spent:
20 hours