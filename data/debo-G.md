## [G-01] Cache array length outside of loop
Description
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.
```txt
2023-08-livepeer/contracts/bonding/BondingVotes.sol::316 => return bondingCheckpoints[_account].startRounds.length > 0;
2023-08-livepeer/contracts/bonding/BondingVotes.sol::341 => } else if (upper < initializedRounds.length) {
2023-08-livepeer/contracts/bonding/BondingVotes.sol::441 => if (startRoundIdx == checkpoints.startRounds.length) {
2023-08-livepeer/contracts/bonding/libraries/SortedArrays.sol::19 => * element (all elements in array are higher than the searched element), the array length is returned.
2023-08-livepeer/contracts/bonding/libraries/SortedArrays.sol::29 => uint256 len = _array.length;
2023-08-livepeer/contracts/bonding/libraries/SortedArrays.sol::65 => if (array.length == 0) {
2023-08-livepeer/contracts/bonding/libraries/SortedArrays.sol::68 => uint256 last = array[array.length - 1];
```
## [G-02] Use not equal to 0 instead of greater than 0 for unsigned integer comparison
Description
When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.
```txt
2023-08-livepeer/contracts/bonding/BondingManager.sol::404 => if (del.bondedAmount > 0) {
2023-08-livepeer/contracts/bonding/BondingManager.sol::578 => } else if (currentBondedAmount > 0 && currentDelegate != _to) {
2023-08-livepeer/contracts/bonding/BondingManager.sol::608 => require(delegationAmount > 0, "delegation amount must be greater than 0");
2023-08-livepeer/contracts/bonding/BondingManager.sol::616 => if (_amount > 0) {
2023-08-livepeer/contracts/bonding/BondingManager.sol::756 => require(_amount > 0, "unbond amount must be greater than 0");
2023-08-livepeer/contracts/bonding/BondingManager.sol::873 => if (treasuryBalanceCeiling > 0) {
2023-08-livepeer/contracts/bonding/BondingManager.sol::875 => if (treasuryBalance >= treasuryBalanceCeiling && nextRoundTreasuryRewardCutRate > 0) {
2023-08-livepeer/contracts/bonding/BondingManager.sol::886 => if (treasuryRewards > 0) {
2023-08-livepeer/contracts/bonding/BondingManager.sol::1160 => return d.delegateAddress == _transcoder && d.bondedAmount > 0;
2023-08-livepeer/contracts/bonding/BondingManager.sol::1171 => return delegators[_delegator].unbondingLocks[_unbondingLockId].withdrawRound > 0;
2023-08-livepeer/contracts/bonding/BondingManager.sol::1276 => // Make sure there is a round to claim i.e. end round - (start round - 1) > 0
2023-08-livepeer/contracts/bonding/BondingVotes.sol::316 => return bondingCheckpoints[_account].startRounds.length > 0;
2023-08-livepeer/contracts/bonding/BondingVotes.sol::331 => if (exactCheckpoint > 0) {
2023-08-livepeer/contracts/bonding/BondingVotes.sol::436 => if (bond.bondedAmount > 0) {
2023-08-livepeer/contracts/bonding/BondingVotes.sol::507 => if (rewardRound > 0) {
```
## [G-03] Long revert strings
Description
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.
```txt
2023-08-livepeer/contracts/bonding/BondingManager.sol::1618 => return ILivepeerToken(controller.getContract(keccak256("LivepeerToken")));
2023-08-livepeer/contracts/bonding/BondingManager.sol::1626 => return IMinter(controller.getContract(keccak256("Minter")));
2023-08-livepeer/contracts/bonding/BondingManager.sol::1634 => return controller.getContract(keccak256("L2Migrator"));
2023-08-livepeer/contracts/bonding/BondingManager.sol::1642 => return IRoundsManager(controller.getContract(keccak256("RoundsManager")));
2023-08-livepeer/contracts/bonding/BondingManager.sol::1646 => return controller.getContract(keccak256("Treasury"));
2023-08-livepeer/contracts/bonding/BondingManager.sol::1650 => return IBondingVotes(controller.getContract(keccak256("BondingVotes")));
2023-08-livepeer/contracts/bonding/BondingManager.sol::1654 => require(msg.sender == controller.getContract(keccak256("TicketBroker")), "caller must be TicketBroker");
2023-08-livepeer/contracts/bonding/BondingManager.sol::1658 => require(msg.sender == controller.getContract(keccak256("RoundsManager")), "caller must be RoundsManager");
2023-08-livepeer/contracts/bonding/BondingManager.sol::1662 => require(msg.sender == controller.getContract(keccak256("Verifier")), "caller must be Verifier");
2023-08-livepeer/contracts/bonding/BondingVotes.sol::540 => return IBondingManager(controller.getContract(keccak256("BondingManager")));
2023-08-livepeer/contracts/bonding/BondingVotes.sol::547 => return IRoundsManager(controller.getContract(keccak256("RoundsManager")));
2023-08-livepeer/contracts/treasury/LivepeerGovernor.sol::102 => return IVotes(controller.getContract(keccak256("BondingVotes")));
2023-08-livepeer/contracts/treasury/LivepeerGovernor.sol::109 => return Treasury(payable(controller.getContract(keccak256("Treasury"))));
2023-08-livepeer/contracts/bonding/BondingManager.sol::493 => require(!roundsManager().currentRoundLocked(), "can't update transcoder params, current round is locked");
2023-08-livepeer/contracts/bonding/BondingManager.sol::503 => "caller can't be active or must have already called reward for the current round"
2023-08-livepeer/contracts/bonding/BondingManager.sol::584 => require(!isRegisteredTranscoder(_owner), "registered transcoders can't delegate towards other addresses");
2023-08-livepeer/contracts/bonding/BondingManager.sol::608 => require(delegationAmount > 0, "delegation amount must be greater than 0");
2023-08-livepeer/contracts/bonding/BondingManager.sol::756 => require(_amount > 0, "unbond amount must be greater than 0");
2023-08-livepeer/contracts/bonding/BondingManager.sol::757 => require(_amount <= del.bondedAmount, "amount is greater than bonded amount");
2023-08-livepeer/contracts/bonding/BondingManager.sol::852 => require(isActiveTranscoder(msg.sender), "caller must be an active transcoder");
2023-08-livepeer/contracts/bonding/BondingManager.sol::855 => "caller has already called reward for the current round"
2023-08-livepeer/contracts/bonding/BondingManager.sol::1179 => require(PreciseMathUtils.validPerc(_cutRate), "_cutRate is invalid precise percentage");
2023-08-livepeer/contracts/bonding/BondingManager.sol::1654 => require(msg.sender == controller.getContract(keccak256("TicketBroker")), "caller must be TicketBroker");
2023-08-livepeer/contracts/bonding/BondingManager.sol::1658 => require(msg.sender == controller.getContract(keccak256("RoundsManager")), "caller must be RoundsManager");
2023-08-livepeer/contracts/bonding/BondingManager.sol::1662 => require(msg.sender == controller.getContract(keccak256("Verifier")), "caller must be Verifier");
2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol::77 => return "support=bravo&quorum=for,abstain,against";
```
## [G-04] Use immutable for openzeppelin access controls roles declarations
Description
⚡️ Only valid for solidity versions <0.6.12 ⚡️

Access roles marked as constant results in computing the keccak256 operation each time the variable is used because assigned operations for constant variables are re-evaluated every time.

Changing the variables to immutable results in computing the hash only once on deployment, leading to gas savings.
```txt
2023-08-livepeer/contracts/bonding/BondingManager.sol::8 => import "../libraries/PreciseMathUtils.sol";
2023-08-livepeer/contracts/bonding/BondingManager.sol::10 => import "./libraries/EarningsPoolLIP36.sol";
2023-08-livepeer/contracts/bonding/BondingManager.sol::17 => import "@openzeppelin/contracts/utils/math/SafeMath.sol";
2023-08-livepeer/contracts/bonding/BondingManager.sol::256 => "withdraw round must be before or equal to the current round"
2023-08-livepeer/contracts/bonding/BondingVotes.sol::4 => import "@openzeppelin/contracts/utils/Arrays.sol";
2023-08-livepeer/contracts/bonding/BondingVotes.sol::5 => import "@openzeppelin/contracts/utils/math/SafeCast.sol";
2023-08-livepeer/contracts/bonding/BondingVotes.sol::8 => import "./libraries/EarningsPoolLIP36.sol";
2023-08-livepeer/contracts/bonding/libraries/EarningsPool.sol::6 => import "@openzeppelin/contracts/utils/math/SafeMath.sol";
2023-08-livepeer/contracts/bonding/libraries/EarningsPoolLIP36.sol::5 => import "../../libraries/PreciseMathUtils.sol";
2023-08-livepeer/contracts/bonding/libraries/EarningsPoolLIP36.sol::7 => import "@openzeppelin/contracts/utils/math/SafeMath.sol";
2023-08-livepeer/contracts/bonding/libraries/SortedArrays.sol::6 => import "@openzeppelin/contracts/utils/Arrays.sol";
2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol::4 => import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol::5 => import "@openzeppelin/contracts-upgradeable/governance/GovernorUpgradeable.sol";
2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol::6 => import "@openzeppelin/contracts-upgradeable/interfaces/IERC5805Upgradeable.sol";
2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol::8 => import "../bonding/libraries/EarningsPool.sol";
2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol::9 => import "../bonding/libraries/EarningsPoolLIP36.sol";
2023-08-livepeer/contracts/treasury/IVotes.sol::4 => import "@openzeppelin/contracts-upgradeable/interfaces/IERC5805Upgradeable.sol";
2023-08-livepeer/contracts/treasury/LivepeerGovernor.sol::4 => import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
2023-08-livepeer/contracts/treasury/LivepeerGovernor.sol::5 => import "@openzeppelin/contracts-upgradeable/governance/GovernorUpgradeable.sol";
2023-08-livepeer/contracts/treasury/LivepeerGovernor.sol::6 => import "@openzeppelin/contracts-upgradeable/governance/extensions/GovernorVotesUpgradeable.sol";
2023-08-livepeer/contracts/treasury/LivepeerGovernor.sol::7 => import "@openzeppelin/contracts-upgradeable/governance/extensions/GovernorVotesQuorumFractionUpgradeable.sol";
2023-08-livepeer/contracts/treasury/LivepeerGovernor.sol::8 => import "@openzeppelin/contracts-upgradeable/governance/extensions/GovernorSettingsUpgradeable.sol";
2023-08-livepeer/contracts/treasury/LivepeerGovernor.sol::9 => import "@openzeppelin/contracts-upgradeable/governance/extensions/GovernorTimelockControlUpgradeable.sol";
2023-08-livepeer/contracts/treasury/LivepeerGovernor.sol::11 => import "../bonding/libraries/EarningsPool.sol";
2023-08-livepeer/contracts/treasury/LivepeerGovernor.sol::12 => import "../bonding/libraries/EarningsPoolLIP36.sol";
2023-08-livepeer/contracts/treasury/LivepeerGovernor.sol::17 => import "./GovernorCountingOverridable.sol";
2023-08-livepeer/contracts/treasury/Treasury.sol::4 => import "@openzeppelin/contracts-upgradeable/governance/TimelockControllerUpgradeable.sol";
```
## [G-05] Use shift or rightleft instead of division or multiplication if possible
Description
A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.

While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.
```txt
2023-08-livepeer/contracts/bonding/BondingManager.sol::32 => uint256 constant MAX_FUTURE_ROUND = 2**256 - 1;
```