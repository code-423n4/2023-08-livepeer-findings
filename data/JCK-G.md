
# Gas Optimizations

| Number | Issue | Instances | Gas |
|--------|-------|-----------|-----|
|[G-01]| Don’t cache value if it is only used once  | 12 |   |
|[G-02]| Use assembly to make more efficient back-to-back calls  | 1 |   |
|[G-03]| Don’t initialize variables with default value  | 4 | 24  |
|[G-04]| require() or revert() statements that check input arguments should be at the top of the function (Also restructured some “if’s”)  | 3 |   |
|[G-05]| Arithmatic operations can be unchecked, since there is no underflow or overflow  | 6 | 510  |
|[G-06]| storage variable can be replaced with calldata variable  | 10 |   |
|[G-07]| Use assembly to hash values more efficiently  | 13 | 1040 |
|[G-08]| Use calldata instead of memory for function arguments that do not get mutated  | 3 | 600  |
|[G-09]| Return values from external calls can be cached to avoid unnecessary call  | 1 |   |
|[G-10]| Move storage pointer to top of function to avoid offset calculation  | 5 | 450  |
|[G-11]| Structs can be packed to use fewer storage slots  | 2 | 4000  |
|[G-12]| Multiple accesses of a mapping/array should use a storage pointer  | 2 | 80  |
|[G-13]| Use functions instead of modifiers  | 4 |   |
|[G-14]| Use assembly to validate msg.sender  | 2 |   |
|[G-15]| Using bools for storage incurs overhead  | 6 | 102600 |
|[G-16]| Combine events to save 2 Glogtopic (375 gas)  | 1 | 375  |
|[G-17]| keccak256() should only need to be called on a specific string literal once  | 13 |   |
|[G-18]| A modifier used only once and not being inherited should be inlined to save gas  | 3 |   |
|[G-19]| Change public function visibility to external  | 11 |   |
|[G-20]| Inverting the condition of an if-else-statement wastes gas  | 3 | 9  |
|[G-21]| internal functions not called by the contract should be removed to save deployment gas | 9 |  |
|[G-22]| Amounts should be checked for 0 before calling a transfer  | 2 |   |
|[G-23]| Do not calculate constants  | 1 |   |
|[G-24]| No need to evaluate all expressions to know if one of them is true  | 1 |   |
|[G-25]| Make 3 event parameters indexed when possible  | 7 | 7000  |

## [G-01] Don’t cache value if it is only used once

If a value is only intended to be used once then it should not be cached. Caching the value will result in unnecessary stack manipulation.

### don't cache they value of proposalSnapshot(_proposalId); because they use only once iside they function

```solidity
file:   contracts/treasury/GovernorCountingOverridable.sol

181    uint256 timepoint = proposalSnapshot(_proposalId);

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L181>

### don't cache they value of transcoderTotalStake(_delegate); because they use only once iside they function

```solidity
file:

1315    uint256 currStake = transcoderTotalStake(_delegate);

1360    uint256 currStake = transcoderTotalStake(_delegate);

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1315>

### don't cache they values of MathUtils.percOf(rewards, treasuryRewardCutRate);, MathUtils.percOf(rewards, earningsPool.transcoderRewardCut); and rewards.sub(transcoderCommissionRewards); because they only used one during they function

```solidity
file:   contracts/bonding/BondingManager.sol

355     uint256 treasuryRewards = MathUtils.percOf(rewards, treasuryRewardCutRate);

358     uint256 transcoderCommissionRewards = MathUtils.percOf(rewards, earningsPool.transcoderRewardCut);

359     uint256 delegatorsRewards = rewards.sub(transcoderCommissionRewards);

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L355>

### don't cache the value of _rewards.sub(transcoderCommissionRewards); because they only used once in they function scope

```solidity
file:    contracts/bonding/BondingManager.sol

1473    uint256 delegatorsRewards = _rewards.sub(transcoderCommissionRewards);

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1473>

### don't cache they value of  _lastClaimRound.add(1); or remove_lastClaimRound.add(1); because they are not used inside they function to save gas

```solidity
file:    contracts/bonding/BondingManager.sol

1506    uint256 startRound = _lastClaimRound.add(1);

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1506>

### don't cache the value of _prevEarningsPool.cumulativeFeeFactor; because they only used once in they function scope

```solidity
file:  contracts/bonding/libraries/EarningsPoolLIP36.sol

23     uint256 prevCumulativeFeeFactor = _prevEarningsPool.cumulativeFeeFactor;

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L23>

### don't cache the value of againstVotes + forVotes; because they only used once in they function scope

```solidity
file:   contracts/treasury/GovernorCountingOverridable.sol

122    uint256 opinionatedVotes = againstVotes + forVotes;

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L122>

### don't cache the value of roundsManager().currentRound(); because they only used once in they function scope

```solidity
file:  contracts/bonding/BondingManager.sol

908       function pendingStake(address _delegator, uint256 _endRound) public view returns (uint256) {
        // Silence unused param compiler warning
        _endRound;

        uint256 endRound = roundsManager().currentRound();
        (uint256 stake, ) = pendingStakeAndFees(_delegator, endRound);
        return stake;
    }

923      function pendingFees(address _delegator, uint256 _endRound) public view returns (uint256) {
        // Silence unused param compiler warning
        _endRound;

        uint256 endRound = roundsManager().currentRound();
        (, uint256 fees) = pendingStakeAndFees(_delegator, endRound);
        return fees;
    }

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L908-L915>

## [G-02] Use assembly to make more efficient back-to-back calls

In the instance below, two external calls, both of which take two function parameters, are performed. We can potentially avoid memory expansion costs by using assembly to utilize scratch space + free memory pointer memory offsets for the function calls. We will use the same memory space for both function calls.

```solidity
file:   contracts/bonding/BondingManager.sol

595             if (currPool.cumulativeRewardFactor == 0) {
             currPool.cumulativeRewardFactor = cumulativeFactorsPool(newDelegate, newDelegate.lastRewardRound)
                 .cumulativeRewardFactor;
         }
         if (currPool.cumulativeFeeFactor == 0) {
             currPool.cumulativeFeeFactor = cumulativeFactorsPool(newDelegate, newDelegate.lastFeeRound)
                 .cumulativeFeeFactor;
         }
```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L595-L602>

## [G-03] Don’t initialize variables with default value

In such cases, initializing the variables with default values would be unnecessary and can be considered a waste of gas. Additionally, initializing variables with default values can sometimes lead to unnecessary storage operations, which can increase gas costs. For example, if you have a large array of variables, initializing them all with default values can result in a lot of unnecessary storage writes, which can increase the gas costs of your contract.

```solidity
file:  contracts/bonding/BondingVotes.sol

373     amount = 0;

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L373>

```solidity
file:  contracts/bonding/BondingManager.sol

773     del.startRound = 0;

1535    t.cumulativeFees = 0;

1536    t.cumulativeRewards = 0;

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L773>

## [G-04] require() or revert() statements that check input arguments should be at the top of the function (Also restructured some “if’s”)

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.

### use require before they intialization of del and lock variable can save gas

```solidity
file:    contracts/bonding/BondingManager.sol

249       function withdrawStake(uint256 _unbondingLockId) external whenSystemNotPaused currentRoundInitialized {
        Delegator storage del = delegators[msg.sender];
        UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];

        require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID"); 

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L249-L253>

### use require before they intialization of del and lock variable can save gas

```solidity
file:   contracts/bonding/BondingManager.sol

1564       function processRebond(
        address _delegator,
        uint256 _unbondingLockId,
        address _newPosPrev,
        address _newPosNext
    ) internal autoCheckpoint(_delegator) {
        Delegator storage del = delegators[_delegator];
        UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];

        require(isValidUnbondingLock(_delegator, _unbondingLockId), "invalid unbonding lock ID");

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1564-L1573>

### use the require statment before the intilization to  del variable

```solidity
file:   contracts/bonding/BondingManager.sol

745        function unbondWithHint(
        uint256 _amount,
        address _newPosPrev,
        address _newPosNext
    ) public whenSystemNotPaused currentRoundInitialized autoClaimEarnings(msg.sender) autoCheckpoint(msg.sender) {
        require(delegatorStatus(msg.sender) == DelegatorStatus.Bonded, "caller must be bonded");

        Delegator storage del = delegators[msg.sender];

        require(_amount > 0, "unbond amount must be greater than 0"); 
```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L745-L754>

## [G-05] Arithmatic operations can be unchecked, since there is no underflow or overflow

```solidity
file:  contracts/treasury/GovernorCountingOverridable.sol

110    uint256 totalVotes = againstVotes + forVotes + abstainVotes;

122    uint256 opinionatedVotes = againstVotes + forVotes;

188    return _weight - _voter.deductions;

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L110>

```solidity
file: contracts/bonding/BondingManager.sol

1599   uint256 startRound = roundsManager().currentRound() + 1;

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1599>

```solidity
file:   contracts/bonding/BondingVotes.sol

182     return getTotalActiveStakeAt(clock() + 1);

195    return getTotalActiveStakeAt(_round + 1);

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L182>

## [G-06] storage variable can be replaced with calldata variable

```solidity
file:  contracts/bonding/BondingManager.sol

1189   function cumulativeFactorsPool(Transcoder storage _transcoder, uint256 _round)

1206   function latestCumulativeFactorsPool(Transcoder storage _transcoder, uint256 _round)

1238   function delegatorCumulativeStakeAndFees(
        Transcoder storage _transcoder,
        uint256 _startRound,
        uint256 _endRound,
        uint256 _stake,
        uint256 _fees
    ) internal view returns (uint256 cStake, uint256 cFees) {

1591      function _checkpointBondingState(
        address _owner,
        Delegator storage _delegator,
        Transcoder storage _transcoder
    ) internal {

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1189>

```solidity
file:   contracts/bonding/BondingVotes.sol

459    function delegatorCumulativeStakeAt(BondingCheckpoint storage bond, uint256 _round)

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L459>

```solidity
file:   contracts/bonding/libraries/EarningsPoolLIP36.sol

18    function updateCumulativeFeeFactor(
        EarningsPool.Data storage earningsPool,
        EarningsPool.Data memory _prevEarningsPool,
        uint256 _fees
    ) internal {

47    function updateCumulativeRewardFactor(
        EarningsPool.Data storage earningsPool,
        EarningsPool.Data memory _prevEarningsPool,
        uint256 _rewards
    ) internal {
```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L18-L22>

```solidity
file:  contracts/bonding/libraries/SortedArrays.sol

28    function findLowerBound(uint256[] storage _array, uint256 _val) internal view returns (uint256) {

64    function pushSorted(uint256[] storage array, uint256 val) internal {

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L28>

```solidity
file:  contracts/treasury/GovernorCountingOverridable.sol

174   function _handleVoteOverrides(
        uint256 _proposalId,
        ProposalTally storage _tally,
        ProposalVoterState storage _voter,
        address _account,
        uint256 _weight
    ) internal returns (uint256) {

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L174-L180>

## [G-07] Use assembly to hash values more efficiently

In the instances below, we can hash values more efficiently by using the least amount of opcodes possible via assembly.

```solidity
file:   contracts/bonding/BondingManager.sol

1616    return ILivepeerToken(controller.getContract(keccak256("LivepeerToken")));

1624    return IMinter(controller.getContract(keccak256("Minter")));

1632    return controller.getContract(keccak256("L2Migrator"));

1640    return IRoundsManager(controller.getContract(keccak256("RoundsManager")));

1644    return controller.getContract(keccak256("Treasury"));

1648    return IBondingVotes(controller.getContract(keccak256("BondingVotes")));

1652    require(msg.sender == controller.getContract(keccak256("TicketBroker")), "caller must be TicketBroker");

1656    require(msg.sender == controller.getContract(keccak256("RoundsManager")), "caller must be RoundsManager");

1660    require(msg.sender == controller.getContract(keccak256("Verifier")), "caller must be Verifier");

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1616>

```solidity
file:  contracts/bonding/BondingVotes.sol

540    return IBondingManager(controller.getContract(keccak256("BondingManager")));

547    return IRoundsManager(controller.getContract(keccak256("RoundsManager")));

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L540>

```solidity
file:  contracts/treasury/LivepeerGovernor.sol

102    return IVotes(controller.getContract(keccak256("BondingVotes")));

109    return Treasury(payable(controller.getContract(keccak256("Treasury"))));

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L102>

## [G-08] Use calldata instead of memory for function arguments that do not get mutated

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```solidity
file:   contracts/treasury/Treasury.sol

16      function initialize(
        uint256 minDelay,
        address[] memory proposers,
        address[] memory executors,
        address admin
    ) external initializer {

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/Treasury.sol#L16-L21>

```solidity
file:    contracts/treasury/LivepeerGovernor.sol

132      function _execute(
        uint256 proposalId,
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        bytes32 descriptionHash
    ) internal

142      function _cancel(
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        bytes32 descriptionHash
    ) internal


```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L132-L138>

## [G-09] Return values from external calls can be cached to avoid unnecessary call

External calls are expensive as they use the STATICCALL/CALL opcode (~100 gas). If you are calling the same external function more than once you should cache the return value to avoid an unnecessary STATICCALL/CALL.

```solidity
file:   contracts/bonding/BondingManager.sol

471    bondingVotes().checkpointTotalActiveStake(currentRoundTotalActiveStake, roundsManager().currentRound());

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L471>

## [G-10] Move storage pointer to top of function to avoid offset calculation

We can avoid unnecessary offset calculations by moving the storage pointer to the top of the function.

### Move BondingCheckpointsByRound storage checkpoints = bondingCheckpoints[_account]; storage pointer to top to save gas 90

```solidity
file:  contracts/bonding/BondingVotes.sol

427     if (_round > clock() + 1) {
            revert FutureLookup(_round, clock() + 1);
        }

        BondingCheckpointsByRound storage checkpoints = bondingCheckpoints[_account];

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L427-L431>

### Move  ProposalTally storage tally = _proposalTallies[_proposalId]; storage pointer to top to save gas

```solidity
file:   contracts/treasury/GovernorCountingOverridable.sol

137      if (_supportInt > uint8(VoteType.Abstain)) {
            revert InvalidVoteType(_supportInt);
        }
        VoteType support = VoteType(_supportInt);

        ProposalTally storage tally = _proposalTallies[_proposalId];

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L137-L142>

### Move Transcoder storage t = transcoders[_transcoder]; storage pointer to top to save gas

```solidity
file:   contracts/bonding/BondingManager.sol

306        ) external whenSystemNotPaused onlyTicketBroker {
        // Silence unused param compiler warning
        _round;

        require(isRegisteredTranscoder(_transcoder), "transcoder must be registered");

        uint256 currentRound = roundsManager().currentRound();

        Transcoder storage t = transcoders[_transcoder];

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L306-L314>

### Move  Transcoder storage t = transcoders[msg.sender]; storage pointer to top to save gas

```solidity
file:   contracts/bonding/BondingManager.sol

490       ) public whenSystemNotPaused currentRoundInitialized {
        require(!roundsManager().currentRoundLocked(), "can't update transcoder params, current round is locked");
        require(MathUtils.validPerc(_rewardCut), "invalid rewardCut percentage");
        require(MathUtils.validPerc(_feeShare), "invalid feeShare percentage");
        require(isRegisteredTranscoder(msg.sender), "transcoder must be registered");

        Transcoder storage t = transcoders[msg.sender];

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L490-L496>

### Move Transcoder storage t = transcoders[del.delegateAddress]; storage pointer to top to save gas

```solidity
file:   contracts/bonding/BondingManager.sol

1505           Delegator storage del = delegators[_delegator];
        uint256 startRound = _lastClaimRound.add(1);
        uint256 currentBondedAmount = del.bondedAmount;
        uint256 currentFees = del.fees;

        // Only will have earnings to claim if you have a delegate
        // If not delegated, skip the earnings claim process
        if (del.delegateAddress != address(0)) {
            (currentBondedAmount, currentFees) = pendingStakeAndFees(_delegator, _endRound);

            // Check whether the endEarningsPool is initialised
            // If it is not initialised set it's cumulative factors so that they can be used when a delegator
            // next claims earnings as the start cumulative factors (see delegatorCumulativeStakeAndFees())
            Transcoder storage t = transcoders[del.delegateAddress];

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1505-L1518>

## [G-11] Structs can be packed to use fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs) we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

Total Instances: 2

Estimated Gas Saved: 2 (slots) * 2000 = 4000

```solidity
file:   contracts/bonding/BondingVotes.sol

24 struct BondingCheckpoint {
        /**
         * @dev The amount of bonded tokens to another delegate as of the lastClaimRound.
         */
        uint256 bondedAmount;
        /**
         * @dev The address of the delegate the account is bonded to. In case of transcoders this is their own address.
         */
        address delegateAddress;
        /**
         * @dev The amount of tokens delegated from delegators to this account. This is only set for transcoders, which
         * have to self-delegate first and then have tokens bonded from other delegators.
         */
        uint256 delegatedAmount;
        /**
         * @dev The last round during which the delegator claimed its earnings. This pegs the value of bondedAmount for
         * rewards calculation in {EarningsPoolLIP36-delegatorCumulativeStakeAndFees}.
         */
        uint256 lastClaimRound;
        /**
         * @dev The last round during which the checkpointed account called {BondingManager-reward}. This is needed to
         * when calculating pending rewards for a delegator to this transcoder, to find the last earning pool available
         * for a given round. In that case we start from the delegator checkpoint and then fetch its delegate address
         * checkpoint as well to find the last earning pool.
         *
         * Notice that this is the only field that comes from the Transcoder struct in BondingManager, not Delegator.
         */
        uint256 lastRewardRound;
    }

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L24-L52>

```solidity
file:  contracts/treasury/GovernorCountingOverridable.sol

37    struct ProposalVoterState {
        bool hasVoted;
        VoteType support;
        // This vote deductions state is only necessary to support the case where a delegator might vote before their
        // transcoder. When that happens, we need to deduct the delegator(s) votes before tallying the transcoder vote.
        uint256 deductions;
    }

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L37-L43>

## [G-12] Multiple accesses of a mapping/array should use a storage pointer

Caching a mapping's value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it.

To achieve this, declare a storage pointer for the variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling stakes[tokenId_], save its reference via a storage pointer: StakeInfo storage stakeInfo = stakes[tokenId_] and use the pointer instead.

Note: These are instances the automated report missed

### Cache storage pointer for earningsPoolPerRound[_round]

```solidity
file:   contracts/bonding/BondingManager.sol

1194      pool.cumulativeRewardFactor = _transcoder.earningsPoolPerRound[_round].cumulativeRewardFactor;

1194      pool.cumulativeFeeFactor = _transcoder.earningsPoolPerRound[_round].cumulativeFeeFactor;

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1194-L1195>

## [G-13] Use functions instead of modifiers

Modifiers:
Modifiers are used to add pre or post-conditions to functions. They are typically applied using the modifier keyword and can be used to enforce access control, check conditions, or perform other checks before or after a function's execution. Modifiers are often used to reduce code duplication and enforce certain rules consistently across multiple functions.

However, using modifiers can lead to increased gas costs, especially when multiple function are applied to a single modifier. Each modifier adds overhead in terms of gas consumption because its code is effectively inserted into the function at the point of the modifier keyword. This results in additional gas usage for each modifier, which can accumulate if multiple modifiers are involved.

Functions:
Functions, on the other hand, are more efficient in terms of gas consumption. When you define a function, its code is written only once and can be called from multiple places within the contract. This reduces gas costs compared to using modifiers since the function's code is not duplicated every time it's called.

By using functions instead of modifiers, you can avoid the gas overhead associated with multiple modifiers and create more gas-efficient smart contracts. If you find that certain logic needs to be applied to multiple functions, you can write a separate function to encapsulate that logic and call it from the functions that require it. This way, you achieve code reusability without incurring unnecessary gas costs.

Let's illustrate the difference between using modifiers and functions with a simple example.

Using Modifiers:

```solidity
pragma solidity ^0.8.0;

contract BankAccountUsingModifiers {
    address public owner;
    uint public balance;

    modifier onlyOwner() {
        require(msg.sender == owner, "Only the owner can perform this action.");
        _;
    }

    constructor() {
        owner = msg.sender;
    }

    function withdraw(uint amount) public onlyOwner {
        require(amount <= balance, "Insufficient balance.");
        balance -= amount;
        // Perform withdrawal logic
    }

    function transfer(address recipient, uint amount) public onlyOwner {
        require(amount <= balance, "Insufficient balance.");
        balance -= amount;
        // Perform transfer logic
    }
}

```

Using Functions:

```solidity
  
pragma solidity ^0.8.0;

contract BankAccountUsingFunctions {
    address public owner;
    uint public balance;

    constructor() {
        owner = msg.sender;
    }

    function onlyOwner() private view returns (bool) {
        return msg.sender == owner;
    }

    function withdraw(uint amount) public {
        require(onlyOwner(), "Only the owner can perform this action.");
        require(amount <= balance, "Insufficient balance.");
        balance -= amount;
        // Perform withdrawal logic
    }

    function transfer(address recipient, uint amount) public {
        require(onlyOwner(), "Only the owner can perform this action.");
        require(amount <= balance, "Insufficient balance.");
        balance -= amount;
        // Perform transfer logic
    }
}

```

In the first contract (BankAccountUsingModifiers), we used a modifier called onlyOwner to check if the caller is the owner of the bank account. The onlyOwner modifier is applied to both the withdraw and transfer functions to ensure that only the owner can perform these actions. However, using the modifier will result in additional gas costs since the modifier's code is duplicated in both functions.

In the second contract (BankAccountUsingFunctions), we replaced the modifier with a private function called onlyOwner, which checks if the caller is the owner. We then call this function at the beginning of both the withdraw and transfer functions. By doing this, we avoid the gas overhead associated with using modifiers, making the contract more gas-efficient.

```solidity
file:  contracts/bonding/BondingManager.sol

124       modifier currentRoundInitialized() {
        _currentRoundInitialized();
        _;
    }

130    modifier autoClaimEarnings(address _delegator) {
        _autoClaimEarnings(_delegator);
        _;
    }

135       modifier autoCheckpoint(address _account) {
        _;
        _checkpointBondingState(_account, delegators[_account], transcoders[_account]);
    }

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L124>

```solidity
file:  contracts/bonding/BondingVotes.sol

87    modifier onlyBondingManager() {
        _onlyBondingManager();
        _;
    }

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L87>

## [G-14] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```solidity
file:  contracts/bonding/BondingManager.sol

559    if (msg.sender != _owner && msg.sender != l2Migrator()) {
    
```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L559>

```solidity
file:  contracts/bonding/BondingVotes.sol

554    if (msg.sender != address(bondingManager())) {

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L554>

## [G-15] Using bools for storage incurs overhead

Both state variables can potentially be set back and forth from true and false. This would result in a Gsset (20000 gas) everytime the values are set to true from false. We can instead use uint(1) in place of true and uint(2) in place of false and pay the Gsset (20000 gas) once during deployment (to set the slot to uint(1). This would save two Gsset (20000 gas). However, a more efficient mitigation would be to pack the variables into a slot with other variables that would inevitably be written to. Please see this finding for a more efficient solution.

```solidity
file:  contracts/bonding/BondingManager.sol
 
1272   bool isTranscoder = _delegator == delegateAddr;

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1272>

```solidity
file:  contracts/bonding/BondingVotes.sol

370    bool isTranscoder = delegateAddress == _account;

398    bool isTranscoder = newDelegate == _account;

399    bool wasTranscoder = previousDelegate == _account;

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L370>

```solidity
file:  contracts/treasury/GovernorCountingOverridable.sol

38   bool hasVoted;

184  bool isTranscoder = _account == delegate;

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L38>

## [G-16] Combine events to save 2 Glogtopic (375 gas)

The events below are only emitted once in the handleRewards function. We can combine the events into one singular event to save two Glogtopic (375 gas) that would otherwise be paid for the additional two events.

```solidity
file:  contracts/bonding/BondingManager.sol

431       emit TranscoderSlashed(_transcoder, _finder, penalty, finderAmount);
            } else {
                // Minter burns the slashed funds
                minter().trustedBurnTokens(burnAmount);

                emit TranscoderSlashed(_transcoder, address(0), penalty, 0);
            }
        } else {
            emit TranscoderSlashed(_transcoder, _finder, 0, 0);

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L431-L439>

## [G-17] keccak256() should only need to be called on a specific string literal once

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once.

```solidity
file:   contracts/bonding/BondingManager.sol

1616    return ILivepeerToken(controller.getContract(keccak256("LivepeerToken")));

1624    return IMinter(controller.getContract(keccak256("Minter")));

1632    return controller.getContract(keccak256("L2Migrator"));

1640    return IRoundsManager(controller.getContract(keccak256("RoundsManager")));

1644    return controller.getContract(keccak256("Treasury"));

1648    return IBondingVotes(controller.getContract(keccak256("BondingVotes")));

1652    require(msg.sender == controller.getContract(keccak256("TicketBroker")), "caller must be TicketBroker");

1656    require(msg.sender == controller.getContract(keccak256("RoundsManager")), "caller must be RoundsManager");

1660    require(msg.sender == controller.getContract(keccak256("Verifier")), "caller must be Verifier");

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1616>

```solidity
file:  contracts/bonding/BondingVotes.sol

540    return IBondingManager(controller.getContract(keccak256("BondingManager")));

547    return IRoundsManager(controller.getContract(keccak256("RoundsManager")));

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L540>

```solidity
file:  contracts/treasury/LivepeerGovernor.sol

102    return IVotes(controller.getContract(keccak256("BondingVotes")));

109    return Treasury(payable(controller.getContract(keccak256("Treasury"))));

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L102>

## [G-18] A modifier used only once and not being inherited should be inlined to save gas

```solidity
file:   contracts/bonding/BondingManager.sol

106     modifier onlyTicketBroker() {
        _onlyTicketBroker();
        _;
    }

112  modifier onlyRoundsManager() {
        _onlyRoundsManager();
        _;
    }

118     modifier onlyVerifier() {
        _onlyVerifier();
        _;
    }
```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L106-L109>

## [G-19] Change public function visibility to external

Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

```solidity
file: contracts/treasury/GovernorCountingOverridable.sol

76    function COUNTING_MODE() public pure virtual override returns (string memory) {

217   function votes() public view virtual returns (IVotes);

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L76>

```solidity
file:   contracts/bonding/BondingManager.sol

679       function transferBond(
        address _delegator,
        uint256 _amount,
        address _oldDelegateNewPosPrev,
        address _oldDelegateNewPosNext,
        address _newDelegateNewPosPrev,
        address _newDelegateNewPosNext
    ) public whenSystemNotPaused currentRoundInitialized {

908  function pendingStake(address _delegator, uint256 _endRound) public view returns (uint256) {

923  function pendingFees(address _delegator, uint256 _endRound) public view returns (uint256) {

946  function transcoderStatus(address _transcoder) public view returns (TranscoderStatus) {

1103 function getTranscoderPoolMaxSize() public view returns (uint256) {

1111 function getTranscoderPoolSize() public view returns (uint256) {

1119 function getFirstTranscoderInPool() public view returns (address) {

1128 function getNextTranscoderInPool(address _transcoder) public view returns (address) {

1136 function getTotalBonded() public view returns (uint256) {

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L679-L686>

## [G‑20] Inverting the condition of an if-else-statement wastes gas

Flipping the true and false blocks instead saves 3 gas

```solidity
file:  contracts/bonding/BondingVotes.sol

98    revert FutureLookup(_round, currentRound == 0 ? 0 : currentRound - 1);

401   uint256 previousDelegateVotes = wasTranscoder ? previous.delegatedAmount : 0;

402   uint256 currentDelegateVotes = isTranscoder ? current.delegatedAmount : 0;

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L98>

## [G‑21] internal functions not called by the contract should be removed to save deployment gas

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

```solidity
file:  contracts/treasury/GovernorCountingOverridable.sol

64    function __GovernorCountingOverridable_init(uint256 _quota) internal onlyInitializing {

107   function _quorumReached(uint256 _proposalId) internal view virtual override returns (bool) {

118   function _voteSucceeded(uint256 _proposalId) internal view virtual override returns (bool) {

130       function _countVote(
        uint256 _proposalId,
        address _account,
        uint8 _supportInt,
        uint256 _weight,
        bytes memory // params
    ) internal virtual override 

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L64>

```solidity
file: contracts/bonding/libraries/SortedArrays.sol

28   function findLowerBound(uint256[] storage _array, uint256 _val) internal view returns (uint256) {

64   function pushSorted(uint256[] storage array, uint256 val) internal {

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L28>

```solidity
file:   contracts/bonding/libraries/EarningsPoolLIP36.sol

18       function updateCumulativeFeeFactor(
        EarningsPool.Data storage earningsPool,
        EarningsPool.Data memory _prevEarningsPool,
        uint256 _fees
    ) internal {

47  function updateCumulativeRewardFactor(
        EarningsPool.Data storage earningsPool,
        EarningsPool.Data memory _prevEarningsPool,
        uint256 _rewards
    ) internal {

71      function delegatorCumulativeStakeAndFees(
        EarningsPool.Data memory _startPool,
        EarningsPool.Data memory _endPool,
        uint256 _stake,
        uint256 _fees
    ) internal pure returns (uint256 cStake, uint256 cFees) {

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L18-L22>

## [G-22] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value check here:

```solidity
file:  contracts/bonding/BondingManager.sol

265   minter().trustedTransferTokens(msg.sender, amount);

185   minter().trustedWithdrawETH(_recipient, _amount);

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L265>

## [G-23] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.
Instances include:

```solidity
file: contracts/bonding/BondingManager.sol

32   uint256 constant MAX_FUTURE_ROUND = 2**256 - 1;

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L32>

## [G-24] No need to evaluate all expressions to know if one of them is true

When we have a code expressionA || expressionB if expressionA is true then expressionB will not be evaluated and gas saved;

```solidity
file:  contracts/bonding/BondingManager.sol

499  require(
            !isActiveTranscoder(msg.sender) || t.lastRewardRound == currentRound,
            "caller can't be active or must have already called reward for the current round"
        );

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L499-L503>

## [G-25] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
file:  contracts/bonding/IBondingManager.sol

9   event TranscoderUpdate(address indexed transcoder, uint256 rewardCut, uint256 feeShare);

10  event TranscoderActivated(address indexed transcoder, uint256 activationRound);

11  event TranscoderDeactivated(address indexed transcoder, uint256 deactivationRound);

13  event Reward(address indexed transcoder, uint256 amount);

14  event TreasuryReward(address indexed transcoder, address treasury, uint256 amount);

38  event WithdrawFees(address indexed delegator, address recipient, uint256 amount);
```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L9>

```solidity
file:  contracts/bonding/IBondingVotes.sol

27    event DelegatorBondedAmountChanged(address indexed delegate, uint256 previousBondedAmount, uint256 newBondedAmount);

```

<https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingVotes.sol#L27>
