https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol

1)Some functions are marked as public when they could be marked as internal to reduce gas costs. For example, the proposalVotes function doesn't need to be accessible externally, and marking it as internal can save gas.