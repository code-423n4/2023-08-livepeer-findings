## Costly redundant operation in _handleVoteOverrides() and _countVote() on storage mapping can be prevented

when **_countVote()** is called, it makes a call to **_handleVoteOverrides()** which handles vote overrides that delegators can make to their corresponding delegated transcoder votes. Within this function, it also makes a costly operation with the **storage** tally mapping which later gets remodified again in the _countVote() as part of the overriding process. However when voters.support is the same as delegateVoter.support, this costly tally modification and remodification are not needed as it just going to cost more.

Below is the suggested revised code to save this redundant operation:

```diff
function _handleVoteOverrides(
        uint256 _proposalId,
        ProposalTally storage _tally,
        ProposalVoterState storage _voter,
        address _account,
        uint256 _weight
+   ) internal returns (uint256, bool) {
-    ) internal returns (uint256) {
        uint256 timepoint = proposalSnapshot(_proposalId);
        address delegate = votes().delegatedAt(_account, timepoint);

        bool isTranscoder = _account == delegate;
        if (isTranscoder) {
            // deduce weight from any previous delegators for this transcoder to
            // make a vote
+            return (_weight - _voter.deductions, false);
-            return _weight - _voter.deductions;
        }

        // this is a delegator, so add a deduction to the delegated transcoder
        ProposalVoterState storage delegateVoter = _tally.voters[delegate];
        delegateVoter.deductions += _weight;

        if (delegateVoter.hasVoted) {
            // this is a delegator overriding its delegated transcoder vote,
            // we need to update the current totals to move the weight of
            // the delegator vote to the right outcome.
            VoteType delegateSupport = delegateVoter.support;
+            if (delegateSupport == _voters.support) return (_weight, true);

            if (delegateSupport == VoteType.Against) {
                _tally.againstVotes -= _weight;
            } else if (delegateSupport == VoteType.For) {
                _tally.forVotes -= _weight;
            } else {
                assert(delegateSupport == VoteType.Abstain);
                _tally.abstainVotes -= _weight;
            }
        }

+        return (_weight, false);
-        return _weight;
    }
```
in _countVotes()
```diff
+  (_weight, bool isSame) = _handleVoteOverrides(_proposalId, tally, voter, _account, _weight);
-  _weight = _handleVoteOverrides(_proposalId, tally, voter, _account, _weight);

+  if (isSame) return;

if (support == VoteType.Against) {
      tally.againstVotes += _weight;
} else if (support == VoteType.For) {
      tally.forVotes += _weight;
} else {
      tally.abstainVotes += _weight;
}
```