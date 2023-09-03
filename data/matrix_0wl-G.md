## Gas Optimizations

|        | Issue                                                                              |
| ------ | :--------------------------------------------------------------------------------- |
| GAS-1  | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE        |
| GAS-2  | Setting the constructor to payable                                                 |
| GAS-3  | USE FUNCTION INSTEAD OF MODIFIERS                                                  |
| GAS-4  | MODIFIERS ARE REDUNDANT IF USED ONLY ONCE OR NOT USED AT ALL                       |
| GAS-5  | Functions guaranteed to revert when called by normal users can be marked payable   |
| GAS-6  | Functions guaranteed to revert when called by normal users can be marked `payable` |
| GAS-7  | Use a more recent version of solidity                                              |
| GAS-8  | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                                |
| GAS-9  | Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead             |
| GAS-10 | USE BYTES32 INSTEAD OF STRING                                                      |

### [GAS-1] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

#### Description:

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything:

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

616:             livepeerToken().transferFrom(msg.sender, address(minter()), _amount);

679:     function transferBond(

```

### [GAS-2] Setting the constructor to payable

#### Description:

Saves ~13 gas per instance

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

149:     constructor(address _controller) Manager(_controller) {}

```

```solidity
File: contracts/bonding/BondingVotes.sol

107:     constructor(address _controller) Manager(_controller) {}

```

```solidity
File: contracts/treasury/LivepeerGovernor.sol

43:     constructor(address _controller) Manager(_controller) {

```

### [GAS-3] USE FUNCTION INSTEAD OF MODIFIERS

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

106:     modifier onlyTicketBroker() {

112:     modifier onlyRoundsManager() {

118:     modifier onlyVerifier() {

124:     modifier currentRoundInitialized() {

130:     modifier autoClaimEarnings(address _delegator) {

135:     modifier autoCheckpoint(address _account) {

```

```solidity
File: contracts/bonding/BondingVotes.sol

87:     modifier onlyBondingManager() {

95:     modifier onlyPastRounds(uint256 _round) {

```

### [GAS-4] MODIFIERS ARE REDUNDANT IF USED ONLY ONCE OR NOT USED AT ALL

#### Description:

https://code4rena.com/reports/2022-09-y2k-finance#g-03-modifiers-are-redundant-if-used-only-once-or-not-used-at-all-5-instances

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

106:     modifier onlyTicketBroker() {

112:     modifier onlyRoundsManager() {

118:     modifier onlyVerifier() {

```

### [GAS-5] Functions guaranteed to revert when called by normal users can be marked payable

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

106:     modifier onlyTicketBroker() {

107:         _onlyTicketBroker();

112:     modifier onlyRoundsManager() {

113:         _onlyRoundsManager();

118:     modifier onlyVerifier() {

119:         _onlyVerifier();

155:     function setUnbondingPeriod(uint64 _unbondingPeriod) external onlyControllerOwner {

167:     function setTreasuryRewardCutRate(uint256 _cutRate) external onlyControllerOwner {

176:     function setTreasuryBalanceCeiling(uint256 _ceiling) external onlyControllerOwner {

186:     function setNumActiveTranscoders(uint256 _numActiveTranscoders) external onlyControllerOwner {

306:     ) external whenSystemNotPaused onlyTicketBroker {

399:     ) external whenSystemNotPaused onlyVerifier autoClaimEarnings(_transcoder) autoCheckpoint(_transcoder) {

462:     function setCurrentRoundTotalActiveStake() external onlyRoundsManager {

1651:     function _onlyTicketBroker() internal view {

1655:     function _onlyRoundsManager() internal view {

1659:     function _onlyVerifier() internal view {

```

```solidity
File: contracts/bonding/BondingVotes.sol

87:     modifier onlyBondingManager() {

88:         _onlyBondingManager();

95:     modifier onlyPastRounds(uint256 _round) {

167:     function getPastVotes(address _account, uint256 _round) external view onlyPastRounds(_round) returns (uint256) {

194:     function getPastTotalSupply(uint256 _round) external view onlyPastRounds(_round) returns (uint256) {

218:     function delegatedAt(address _account, uint256 _round) external view onlyPastRounds(_round) returns (address) {

266:     ) external virtual onlyBondingManager {

303:     function checkpointTotalActiveStake(uint256 _totalStake, uint256 _round) external virtual onlyBondingManager {

553:     function _onlyBondingManager() internal view {

```

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

64:     function __GovernorCountingOverridable_init(uint256 _quota) internal onlyInitializing {

68:     function __GovernorCountingOverridable_init_unchained(uint256 _quota) internal onlyInitializing {

```

### [GAS-6] Functions guaranteed to revert when called by normal users can be marked `payable`

#### Description:

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
If a function modifier such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

155:     function setUnbondingPeriod(uint64 _unbondingPeriod) external onlyControllerOwner {

167:     function setTreasuryRewardCutRate(uint256 _cutRate) external onlyControllerOwner {

176:     function setTreasuryBalanceCeiling(uint256 _ceiling) external onlyControllerOwner {

186:     function setNumActiveTranscoders(uint256 _numActiveTranscoders) external onlyControllerOwner {

462:     function setCurrentRoundTotalActiveStake() external onlyRoundsManager {


```

```solidity
File: contracts/bonding/BondingVotes.sol

167:     function getPastVotes(address _account, uint256 _round) external view onlyPastRounds(_round) returns (uint256) {

194:     function getPastTotalSupply(uint256 _round) external view onlyPastRounds(_round) returns (uint256) {

218:     function delegatedAt(address _account, uint256 _round) external view onlyPastRounds(_round) returns (address) {

303:     function checkpointTotalActiveStake(uint256 _totalStake, uint256 _round) external virtual onlyBondingManager {

553:     function _onlyBondingManager() internal view {

```

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

64:     function __GovernorCountingOverridable_init(uint256 _quota) internal onlyInitializing {

68:     function __GovernorCountingOverridable_init_unchained(uint256 _quota) internal onlyInitializing {

```

### [GAS-7] Use a more recent version of solidity

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/BondingVotes.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/IBondingManager.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/IBondingVotes.sol

2: pragma solidity ^0.8.9;

```

```solidity
File: contracts/bonding/libraries/EarningsPoolLIP36.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/bonding/libraries/SortedArrays.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/IVotes.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/LivepeerGovernor.sol

2: pragma solidity 0.8.9;

```

```solidity
File: contracts/treasury/Treasury.sol

2: pragma solidity 0.8.9;

```

### [GAS-8] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.
Note that this optimization seems to be dependent on usage of a more recent Solidity version. The following gas savings are on version 0.8.17.

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

564:         if (delegatorStatus(_owner) == DelegatorStatus.Unbonded) {
                require(_to != _owner, "INVALID_DELEGATE");
            } else {
                require(currentDelegate == _to, "INVALID_DELEGATE_CHANGE");
            }

```

### [GAS-9] Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead

#### Description:

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Use a larger size then downcast where needed.

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingManager.sol

35:     uint64 public unbondingPeriod;

155:     function setUnbondingPeriod(uint64 _unbondingPeriod) external onlyControllerOwner {

```

```solidity
File: contracts/bonding/BondingVotes.sol

129:     function decimals() external pure returns (uint8) {

137:     function clock() public view returns (uint48) {

237:         uint8,

```

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

22:     error InvalidVoteType(uint8 voteType);

133:         uint8 _supportInt,

137:         if (_supportInt > uint8(VoteType.Abstain)) {

```

```solidity
File: contracts/treasury/IVotes.sol

17:     function decimals() external view returns (uint8);

```

### [GAS-10] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: contracts/bonding/BondingVotes.sol

115:     function name() external pure returns (string memory) {

122:     function symbol() external pure returns (string memory) {

145:     function CLOCK_MODE() external pure returns (string memory) {

```

```solidity
File: contracts/bonding/IBondingVotes.sol

20:     error MustCallBondingManager(string bondingManagerFunction);

```

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

76:     function COUNTING_MODE() public pure virtual override returns (string memory) {

```

```solidity
File: contracts/treasury/IVotes.sol

13:     function name() external view returns (string memory);

15:     function symbol() external view returns (string memory);

```
