## [L-01] `quota` parameter of `LivepeerGovernor.sol#initialize()` is shadowed by `GovernorCountingOverridable.quota`

File: https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/LivepeerGovernor.sol#L71

```
function initialize(
        uint256 initialVotingDelay,
        uint256 initialVotingPeriod,
        uint256 initialProposalThreshold,
        uint256 initialQuorum,
        uint256 quota
    ) public initializer {
        __Governor_init("LivepeerGovernor");
        __GovernorSettings_init(initialVotingDelay, initialVotingPeriod, initialProposalThreshold);
        __GovernorTimelockControl_init(treasury());

        // The GovernorVotes module will hold a fixed reference to the votes contract. If we ever change its address we
        // need to call the {bumpGovernorVotesTokenAddress} function to update it in here as well.
        __GovernorVotes_init(votes());

        __GovernorVotesQuorumFraction_init(initialQuorum);

        __GovernorCountingOverridable_init(quota);//@audit variable shadowing.
    }
```

#### Recommended Mitigation Step
Rename `quota` parameter of the initialize function of LivepeerGovernor.sol to avoid variable shadowing with `GovernorCountingOverridable.quota`