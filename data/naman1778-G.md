## [G-01] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.

    File: contracts/bonding/BondingManager.sol	

    1271: address delegateAddr = del.delegateAddress;

    1272: bool isTranscoder = _delegator == delegateAddr;

    1571: UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol

    File: contracts/bonding/BondingVotes.sol	

    343: uint256 nextInitedRound = initializedRounds[upper];

    370: bool isTranscoder = delegateAddress == _account;

    398: bool isTranscoder = newDelegate == _account;

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol

    File: contracts/treasury/GovernorCountingOverridable.sol	

    184: bool isTranscoder = _account == delegate;

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }

    contract Contract0 {
        uint256 num = 5;
        uint256 sum;
        function not_optimized() public returns(bool){
            uint256 num1 = num;
            sum = num1 + 2;
        }
    }

    contract Contract1 {
        uint256 num = 5;
        uint256 sum;
        function optimized() public returns(bool){
            sum = num + 2;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63799                                     | 244             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24462           | 24462 | 24462  | 24462 | 1       |


| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63599                                     | 243             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24460           | 24460 | 24460  | 24460 | 1       |

## [G-02] Revert Transaction as soon as possible

Always try reverting transactions as early as possible when using require statements . In case a transaction revert occurs, the user will pay the gas up until the revert was executed

    File: contracts/bonding/BondingManager.sol	

    249: function withdrawStake(uint256 _unbondingLockId) external whenSystemNotPaused currentRoundInitialized {
    250:     Delegator storage del = delegators[msg.sender];
    251:     UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];
    252: 
    253:     require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID");

    745: function unbondWithHint(
    746:     uint256 _amount,
    747:     address _newPosPrev,
    748:     address _newPosNext
    749: ) public whenSystemNotPaused currentRoundInitialized autoClaimEarnings(msg.sender) autoCheckpoint(msg.sender) {
    750:     require(delegatorStatus(msg.sender) == DelegatorStatus.Bonded, "caller must be bonded");
    752:     Delegator storage del = delegators[msg.sender];
    754:     require(_amount > 0, "unbond amount must be greater than 0");

    842: function rewardWithHint(address _newPosPrev, address _newPosNext)
    843:     public
    844:     whenSystemNotPaused
    845:     currentRoundInitialized
    846:     autoCheckpoint(msg.sender)
    847: {
    848:     uint256 currentRound = roundsManager().currentRound();
    849: 
    850:     require(isActiveTranscoder(msg.sender), "caller must be an active transcoder");

    1564: function processRebond(
    1565:     address _delegator,
    1566:     uint256 _unbondingLockId,
    1567:     address _newPosPrev,
    1568:     address _newPosNext
    1569: ) internal autoCheckpoint(_delegator) {
    1570:     Delegator storage del = delegators[_delegator];
    1571:     UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];
    1572: 
    1573:     require(isValidUnbondingLock(_delegator, _unbondingLockId), "invalid unbonding lock ID");

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol

