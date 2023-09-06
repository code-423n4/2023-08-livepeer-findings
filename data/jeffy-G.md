## [G-01] use if + revert instead of require/assert to save gas
Consider using custom errors instead of revert strings. Can save gas when the revert condition has been met during runtime.
Here are some examples of where this optimization could be used:
```
File: contracts/bonding/BondingManager.sol
253: require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID");
254:  require(
            lock.withdrawRound <= roundsManager().currentRound(),
            "withdraw round must be before or equal to the current round"
        );
279: require(_recipient != address(0), "invalid recipient");
281: require(fees >= _amount, "insufficient fees to withdraw");
310: require(isRegisteredTranscoder(_transcoder), "transcoder must be registered");
491: require(!roundsManager().currentRoundLocked(), "can't update transcoder params, current round is locked");
492: require(MathUtils.validPerc(_rewardCut), "invalid rewardCut percentage");
493: require(MathUtils.validPerc(_feeShare), "invalid feeShare percentage");
494: require(isRegisteredTranscoder(msg.sender), "transcoder must be registered");
499: require(
            !isActiveTranscoder(msg.sender) || t.lastRewardRound == currentRound,
            "caller can't be active or must have already called reward for the current round"
        );
563: require(_to != _owner, "INVALID_DELEGATE");
565: require(currentDelegate == _to, "INVALID_DELEGATE_CHANGE");
582: require(!isRegisteredTranscoder(_owner), "registered transcoders can't delegate towards other addresses");
606: require(delegationAmount > 0, "delegation amount must be greater than 0");
722: require(oldDelDelegate != _delegator, "INVALID_DELEGATOR");
750: require(delegatorStatus(msg.sender) == DelegatorStatus.Bonded, "caller must be bonded");
754: require(_amount > 0, "unbond amount must be greater than 0");
755: require(_amount <= del.bondedAmount, "amount is greater than bonded amount");
801: require(delegatorStatus(msg.sender) != DelegatorStatus.Unbonded, "caller must be bonded");
824: require(delegatorStatus(msg.sender) == DelegatorStatus.Unbonded, "caller must be unbonded");
850: require(isActiveTranscoder(msg.sender), "caller must be an active transcoder");
851: require(
            transcoders[msg.sender].lastRewardRound != currentRound,
            "caller has already called reward for the current round"
        );
1177: require(PreciseMathUtils.validPerc(_cutRate), "_cutRate is invalid precise percentage");
1573: require(isValidUnbondingLock(_delegator, _unbondingLockId), "invalid unbonding lock ID");
1652: require(msg.sender == controller.getContract(keccak256("TicketBroker")), "caller must be TicketBroker");
1656: require(msg.sender == controller.getContract(keccak256("RoundsManager")), "caller must be RoundsManager");
1660: require(msg.sender == controller.getContract(keccak256("Verifier")), "caller must be Verifier");
1664: require(roundsManager().currentRoundInitialized(), "current round is not initialized");

File: contracts/bonding/libraries/SortedArrays.sol
41: assert(upperIdx < len);

File: contracts/treasury/GovernorCountingOverridable.sol
206: assert(delegateSupport == VoteType.Abstain);

```
 - BondingManager.sol : [[253](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L253), [254](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L254), [279](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L279), [281](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L281), [310](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L310), [491](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L491), [492](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L492), [493](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L493), [494](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L494), [499](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L499), [563](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L563), [565](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L565), [582](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L582), [606](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L606), [722](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L722), [750](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L750), [754](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L754), [755](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L755), [801](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L801), [824](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L824), [850](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L850), [851](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L851), [1177](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L1177), [1573](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L1573), [1652](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L1652), [1656](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L1656), [1660](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L1660), [1664](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L1664)]
 - SortedArrays.sol : [[41](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/libraries/SortedArrays.sol#L41)]
 - GovernorCountingOverridable.sol : [[206](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L206)]

Note that this will only decrease runtime gas when the revert condition has been met. Regardless, it will decrease deploy time gas

## [G-02] require statements using revert strings bigger then 32 bytes consume more gas

```
File: contracts/bonding/BondingManager.sol
254:  require(
            lock.withdrawRound <= roundsManager().currentRound(),
            "withdraw round must be before or equal to the current round"
        );
491: require(!roundsManager().currentRoundLocked(), "can't update transcoder params, current round is locked");
499: require(
            !isActiveTranscoder(msg.sender) || t.lastRewardRound == currentRound,
            "caller can't be active or must have already called reward for the current round"
        );
582: require(!isRegisteredTranscoder(_owner), "registered transcoders can't delegate towards other addresses");
606: require(delegationAmount > 0, "delegation amount must be greater than 0");
755: require(_amount <= del.bondedAmount, "amount is greater than bonded amount");
850: require(isActiveTranscoder(msg.sender), "caller must be an active transcoder");
851: require(
            transcoders[msg.sender].lastRewardRound != currentRound,
            "caller has already called reward for the current round"
        );
1177: require(PreciseMathUtils.validPerc(_cutRate), "_cutRate is invalid precise percentage");
```

- BondingManager.sol : [[254](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L254), [491](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L491), [499](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L499), [582](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L582), [606](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L606)]


## [G-03] mark event arguments up to 3 arguments as indexed to save gas
Using this keyword with value types such as `uint`, `bool`, and `address` saves gas; however, this is only true for these value types, since indexing `bytes`, `strings` and `arrays` is more costly than not indexing them.

Here are some examples of where this optimization could be used:

```
File: contracts/bonding/IBondingManager.sol
9: event TranscoderUpdate(address indexed transcoder, uint256 rewardCut, uint256 feeShare);
10: event TranscoderActivated(address indexed transcoder, uint256 activationRound);
11: event TranscoderDeactivated(address indexed transcoder, uint256 deactivationRound);
12: event TranscoderSlashed(address indexed transcoder, address finder, uint256 penalty, uint256 finderReward);
13: event Reward(address indexed transcoder, uint256 amount);
14: event TreasuryReward(address indexed transcoder, address treasury, uint256 amount);
15: event Bond(
        address indexed newDelegate,
        address indexed oldDelegate,
        address indexed delegator,
        uint256 additionalAmount,
        uint256 bondedAmount
    );
22: event Unbond(
        address indexed delegate,
        address indexed delegator,
        uint256 unbondingLockId,
        uint256 amount,
        uint256 withdrawRound
    );
29: event Rebond(address indexed delegate, address indexed delegator, uint256 unbondingLockId, uint256 amount);
30: event TransferBond(
37: event WithdrawStake(address indexed delegator, uint256 unbondingLockId, uint256 amount, uint256 withdrawRound);
38: event WithdrawFees(address indexed delegator, address recipient, uint256 amount);
39: event EarningsClaimed(

File: contracts/bonding/IBondingVotes.sol
27: event DelegatorBondedAmountChanged(address indexed delegate, uint256 previousBondedAmount, uint256 newBondedAmount);

```
 - IBondingManager.sol : [[9](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L9), [10](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L10), [11](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L11), [12](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L12), [13](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L13), [14](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L14), [15](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L15), [22](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L22), [29](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L29), [30](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L30), [37](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L37), [38](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L38), [39](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L39)]
 - IBondingVotes.sol : [[27](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingVotes.sol#L27)]

## [G-04] <x> += <y> costs more gas than <x> = <x> + <y> for state variables
Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

Here are some examples of where this optimization could be used:

```
File: contracts/treasury/GovernorCountingOverridable.sol
154: tally.againstVotes += _weight;
156: tally.forVotes += _weight;
158: tally.abstainVotes += _weight;
193: delegateVoter.deductions += _weight;
202: _tally.againstVotes -= _weight;
204: _tally.forVotes -= _weight;
207: _tally.abstainVotes -= _weight;

```
 - GovernorCountingOverridable.sol : [[154](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L154), [156](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L156), [158](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L158), [193](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L193), [202](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L202), [204](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L204), [207](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L207)]