### GAS-1 Bools more expensive than uint's for storage variables

There are instances were gas savings can be made by not using books

[struct ProposalVoterState { bool hasVoted; VoteType support; uint256 deductions; }](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L38) GovernorCountingOverridable.sol# line 38 struct uses bool for hasVoted flag

**Impact** -> bools are more expensive than uints because the write operations emits extra SLOAD that firstly reads the slot contents and then goes on to replace the bits the boolean has taken up, it then proceeeds to write this back.

**Recommendation** -> e.g use uint(0) false uint(1) true or uint(1) false uint(2) false instead etc

