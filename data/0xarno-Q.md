
# GovernorCountingOverridable._voteSucceeded(): Confusing code comments deviates from function logic

#### Impact

The code comments are unclear and do not match the actual code logic.

```
     /**
     * @notice The required percentage of "for" votes in relation to the total opinionated votes (for and abstain) for
     * a proposal to succeed. Represented as a MathUtils percentage value (e.g. 6 decimal places).
     */
    uint256 public quota;

 function _voteSucceeded(uint256 _proposalId) internal view virtual override returns (bool) {
        (uint256 againstVotes, uint256 forVotes, ) = proposalVotes(_proposalId);

        // we ignore abstain votes for vote succeeded calculation
        uint256 opinionatedVotes = againstVotes + forVotes;
        return forVotes >= MathUtils.percOf(opinionatedVotes, quota);
    }

```
#### Recommendation

Adjust code comments to follow function logic.

