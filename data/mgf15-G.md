|ID|Title|Category|Severity|Instances|
|-|:-|:-:|:-:|:-:|
| [[G-1](#g-1--use-assembly-to-check-for-address0)] | Use assembly to check for `address(0)` | Gas Optimization | Informational | 4 |
| [[G-2](#g-2--functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable)] | Functions guaranteed to revert when called by normal users can be marked payable | Gas Optimization | Informational | 11 |
| [[G-3](#g-3--long-revert-strings)] | Long revert strings | Gas Optimization | Informational | 10 |
| [[G-4](#g-4-->=-costs-less-gas-than->)] | `>=` costs less gas than `>` | Gas Optimization | Informational | 14 |
| [[G-5](#g-5--x-+=-yx--=-y-costs-more-gas-than-x-=-x-+-yx-=-x---y-for-state-variables)] | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables | Gas Optimization | Informational | 4 |
| [[G-6](#g-6--usage-of-uintsints-smaller-than-32-bytes-256-bits-incurs-overhead)] | Usage of `uints/ints` smaller than 32 bytes (256 bits) incurs overhead | Gas Optimization | Informational | 8 |
| [[G-7](#g-7--use-nested-if-and,-avoid-multiple-check-combinations)] | Use nested if and, avoid multiple check combinations | Gas Optimization | Informational | 7 |
| [[G-8](#g-8--inverting-the-condition-of-an-if-else-statement-wastes-gas)] | Inverting the condition of an if-else-statement wastes gas | Gas Optimization | Informational | 3 |
| [[G-9](#g-9--gas-saving-is-achieved-by-removing-the-delete-keyword-~60k)] | Gas saving is achieved by removing the `delete` keyword (~60k) | Gas Optimization | Informational | 2 |
| [[G-10](#g-10--use-double-if-statements-instead-of-&&)] | Use double `if` statements instead of `&&` | Gas Optimization | Informational | 7 |
| [[G-11](#g-11--use-!=-0-instead-of->-0-for-unsigned-integer-comparison)] | Use `!= 0` instead of `> 0` for unsigned integer comparison | Gas Optimization | Informational | 15 |
| [[G-12](#g-12--use-shift-rightleft-instead-of-divisionmultiplication-if-possible)] | Use shift Right/Left instead of division/multiplication if possible | Gas Optimization | Informational | 1 |
| [[G-13](#g-13--constructors-can-be-marked-payable)] | Constructors can be marked `payable` | Gas Optimization | Informational | 10 |
| [[G-14](#g-14--don't-use-safemath-once-the-solidity-version-is-080-or-greater)] | Don't use `SafeMath` once the solidity version is 0.8.0 or greater | Gas Optimization | Informational | 2 |
| [[G-15](#g-15--reduce-gas-usage-by-moving-to-solidity-0819-or-later)] | Reduce gas usage by moving to Solidity 0.8.19 or later | Gas Optimization | Informational | 10 |
| [[G-16](#g-16--gas-savings-can-be-achieved-by-changing-the-model-for-assigning-value-to-the-structure)] | Gas savings can be achieved by changing the model for assigning value to the structure | Gas Optimization | Informational | 2 |
| [[G-17](#g-17--use-bytes32-instead-of-string)] | USE BYTES32 INSTEAD OF STRING | Gas Optimization | Informational | 7 |
| [[G-18](#g-18--part-of-the-code-can-be-pre-calculated)] | Part of the code can be pre calculated | Gas Optimization | Informational | 3 |
| [[G-19](#g-19---multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct,-where-appropriate)] |  Multiple `address/ID` mappings can be combined into a single mapping of an address/ID to a struct, where appropriate | Gas Optimization | Informational | 9 |
| [[G-20](#g-20--ternary-operation-is-cheaper-than-if-else-statement)] | Ternary operation is cheaper than if-else statement | Gas Optimization | Informational | 10 |

## **G-1** | Use assembly to check for `address(0)`

### **Description** :-

Saves 6 gas per instance if using assembly to check for address(0)

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
279:        require(_recipient != address(0)
```
#### **Context**:-
[BondingManager.sol#L279](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L279)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
424:            if (_finder != address(0)
```
#### **Context**:-
[BondingManager.sol#L424](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L424)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
719:        if (newDel.delegateAddress == address(0)
```
#### **Context**:-
[BondingManager.sol#L719](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L719)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
1512:        if (del.delegateAddress != address(0)
```
#### **Context**:-
[BondingManager.sol#L1512](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1512)

## **G-2** | Functions guaranteed to revert when called by normal users can be marked payable

### **Description** :-

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
155:function setUnbondingPeriod(uint64 _unbondingPeriod) external onlyControllerOwner {
```
#### **Context**:-
[BondingManager.sol#L155](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L155)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
167:function setTreasuryRewardCutRate(uint256 _cutRate) external onlyControllerOwner {
```
#### **Context**:-
[BondingManager.sol#L167](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L167)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
176:function setTreasuryBalanceCeiling(uint256 _ceiling) external onlyControllerOwner {
```
#### **Context**:-
[BondingManager.sol#L176](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L176)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
186:function setNumActiveTranscoders(uint256 _numActiveTranscoders) external onlyControllerOwner {
```
#### **Context**:-
[BondingManager.sol#L186](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L186)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
462:function setCurrentRoundTotalActiveStake() external onlyRoundsManager {
```
#### **Context**:-
[BondingManager.sol#L462](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L462)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
167:function getPastVotes(address _account, uint256 _round) external view onlyPastRounds(_round) returns (uint256) {
```
#### **Context**:-
[BondingVotes.sol#L167](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L167)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
194:function getPastTotalSupply(uint256 _round) external view onlyPastRounds(_round) returns (uint256) {
```
#### **Context**:-
[BondingVotes.sol#L194](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L194)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
218:function delegatedAt(address _account, uint256 _round) external view onlyPastRounds(_round) returns (address) {
```
#### **Context**:-
[BondingVotes.sol#L218](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L218)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
303:function checkpointTotalActiveStake(uint256 _totalStake, uint256 _round) external virtual onlyBondingManager {
```
#### **Context**:-
[BondingVotes.sol#L303](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L303)
```solidity
File:2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol
```
```solidity
64:function __GovernorCountingOverridable_init(uint256 _quota) internal onlyInitializing {
```
#### **Context**:-
[GovernorCountingOverridable.sol#L64](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L64)
```solidity
File:2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol
```
```solidity
68:function __GovernorCountingOverridable_init_unchained(uint256 _quota) internal onlyInitializing {
```
#### **Context**:-
[GovernorCountingOverridable.sol#L68](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L68)

## **G-3** | Long revert strings

### **Description** :-

Revert strings that are longer than 32 bytes require at least one additional mstore, along with additional overhead for computing memory offset, etc.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
491:require(!roundsManager().currentRoundLocked(), "can't update transcoder params, current round is locked")
```
#### **Context**:-
[BondingManager.sol#L491](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L491)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
582:require(!isRegisteredTranscoder(_owner), "registered transcoders can't delegate towards other addresses")
```
#### **Context**:-
[BondingManager.sol#L582](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L582)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
606:require(delegationAmount > 0, "delegation amount must be greater than 0")
```
#### **Context**:-
[BondingManager.sol#L606](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L606)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
754:require(_amount > 0, "unbond amount must be greater than 0")
```
#### **Context**:-
[BondingManager.sol#L754](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L754)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
755:require(_amount <= del.bondedAmount, "amount is greater than bonded amount")
```
#### **Context**:-
[BondingManager.sol#L755](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L755)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
850:require(isActiveTranscoder(msg.sender), "caller must be an active transcoder")
```
#### **Context**:-
[BondingManager.sol#L850](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L850)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
1177:require(PreciseMathUtils.validPerc(_cutRate), "_cutRate is invalid precise percentage")
```
#### **Context**:-
[BondingManager.sol#L1177](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1177)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
1652:require(msg.sender == controller.getContract(keccak256("TicketBroker")), "caller must be TicketBroker")
```
#### **Context**:-
[BondingManager.sol#L1652](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1652)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
1656:require(msg.sender == controller.getContract(keccak256("RoundsManager")), "caller must be RoundsManager")
```
#### **Context**:-
[BondingManager.sol#L1656](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1656)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
1660:require(msg.sender == controller.getContract(keccak256("Verifier")), "caller must be Verifier")
```
#### **Context**:-
[BondingManager.sol#L1660](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1660)

## **G-4** | `>=` costs less gas than `>`

### **Description** :-

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
327: > l
```
#### **Context**:-
[BondingManager.sol#L327](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L327)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
331: < c
```
#### **Context**:-
[BondingManager.sol#L331](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L331)
```solidity
714: < c
```
#### **Context**:-
[BondingManager.sol#L714](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L714)
```solidity
867: < c
```
#### **Context**:-
[BondingManager.sol#L867](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L867)
```solidity
1331: < c
```
#### **Context**:-
[BondingManager.sol#L1331](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1331)
```solidity
1374: < c
```
#### **Context**:-
[BondingManager.sol#L1374](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1374)
```solidity
1670: < c
```
#### **Context**:-
[BondingManager.sol#L1670](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1670)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
316: > 0
```
#### **Context**:-
[BondingVotes.sol#L316](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L316)
```solidity
331: > 0
```
#### **Context**:-
[BondingVotes.sol#L331](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L331)
```solidity
436: > 0
```
#### **Context**:-
[BondingVotes.sol#L436](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L436)
```solidity
507: > 0
```
#### **Context**:-
[BondingVotes.sol#L507](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L507)
```solidity
File:2023-08-livepeer/contracts/bonding/libraries/SortedArrays.sol
```
```solidity
41: < l
```
#### **Context**:-
[SortedArrays.sol#L41](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L41)
```solidity
71: < l
```
#### **Context**:-
[SortedArrays.sol#L71](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L71)
```solidity
File:2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol
```
```solidity
137: > u
```
#### **Context**:-
[GovernorCountingOverridable.sol#L137](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L137)

## **G-5** | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables

### **Description** :-

Using the addition operator instead of plus-equals saves 113 gas

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol
```
```solidity
154:+=
```
#### **Context**:-
[GovernorCountingOverridable.sol#L154](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L154)
```solidity
156:+=
```
#### **Context**:-
[GovernorCountingOverridable.sol#L156](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L156)
```solidity
158:+=
```
#### **Context**:-
[GovernorCountingOverridable.sol#L158](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L158)
```solidity
193:+=
```
#### **Context**:-
[GovernorCountingOverridable.sol#L193](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L193)

## **G-6** | Usage of `uints/ints` smaller than 32 bytes (256 bits) incurs overhead

### **Description** :-

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
35:uint64
```
#### **Context**:-
[BondingManager.sol#L35](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L35)
```solidity
155:uint64
```
#### **Context**:-
[BondingManager.sol#L155](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L155)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
129:uint8
```
#### **Context**:-
[BondingVotes.sol#L129](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L129)
```solidity
237:uint8
```
#### **Context**:-
[BondingVotes.sol#L237](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L237)
```solidity
File:2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol
```
```solidity
22:uint8
```
#### **Context**:-
[GovernorCountingOverridable.sol#L22](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L22)
```solidity
133:uint8
```
#### **Context**:-
[GovernorCountingOverridable.sol#L133](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L133)
```solidity
137:uint8
```
#### **Context**:-
[GovernorCountingOverridable.sol#L137](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L137)
```solidity
File:2023-08-livepeer/contracts/treasury/IVotes.sol
```
```solidity
17:uint8
```
#### **Context**:-
[IVotes.sol#L17](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/IVotes.sol#L17)

## **G-7** | Use nested if and, avoid multiple check combinations

### **Description** :-

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
343:if (prevEarningsPool.cumulativeRewardFactor == 0 &&
```
#### **Context**:-
[BondingManager.sol#L343](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L343)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
559:if (msg.sender != _owner &&
```
#### **Context**:-
[BondingManager.sol#L559](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L559)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
576:if (currentBondedAmount > 0 &&
```
#### **Context**:-
[BondingManager.sol#L576](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L576)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
719:if (newDel.delegateAddress == address(0) &&
```
#### **Context**:-
[BondingManager.sol#L719](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L719)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
873:if (treasuryBalance >= treasuryBalanceCeiling &&
```
#### **Context**:-
[BondingManager.sol#L873](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L873)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
1215:if (pool.cumulativeRewardFactor == 0 &&
```
#### **Context**:-
[BondingManager.sol#L1215](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1215)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
1221:if (pool.cumulativeFeeFactor == 0 &&
```
#### **Context**:-
[BondingManager.sol#L1221](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1221)

## **G-8** | Inverting the condition of an if-else-statement wastes gas

### **Description** :-

Flipping the `true` and `false` blocks instead saves 3 gas

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
98:== 0 ?
```
#### **Context**:-
[BondingVotes.sol#L98](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L98)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
401:= wasTranscoder ?
```
#### **Context**:-
[BondingVotes.sol#L401](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L401)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
402:= isTranscoder ?
```
#### **Context**:-
[BondingVotes.sol#L402](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L402)

## **G-9** | Gas saving is achieved by removing the `delete` keyword (~60k)

### **Description** :-

30k gas savings were made by removing the `delete` keyword. The reason for using the `delete` keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the `delete` key word is unnecessary. If it is removed, around 30k gas savings will be achieved.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
262: delete del.unbondingLocks[_unbondingLockId];
```
#### **Context**:-
[BondingManager.sol#L262](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L262)
```solidity
1580: delete del.unbondingLocks[_unbondingLockId];
```
#### **Context**:-
[BondingManager.sol#L1580](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1580)

## **G-10** | Use double `if` statements instead of `&&`

### **Description** :-

If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
343:if (prevEarningsPool.cumulativeRewardFactor == 0 &&
```
#### **Context**:-
[BondingManager.sol#L343](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L343)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
559:if (msg.sender != _owner &&
```
#### **Context**:-
[BondingManager.sol#L559](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L559)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
576:if (currentBondedAmount > 0 &&
```
#### **Context**:-
[BondingManager.sol#L576](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L576)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
719:if (newDel.delegateAddress == address(0) &&
```
#### **Context**:-
[BondingManager.sol#L719](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L719)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
873:if (treasuryBalance >= treasuryBalanceCeiling &&
```
#### **Context**:-
[BondingManager.sol#L873](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L873)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
1215:if (pool.cumulativeRewardFactor == 0 &&
```
#### **Context**:-
[BondingManager.sol#L1215](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1215)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
1221:if (pool.cumulativeFeeFactor == 0 &&
```
#### **Context**:-
[BondingManager.sol#L1221](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1221)

## **G-11** | Use `!= 0` instead of `> 0` for unsigned integer comparison

### **Description** :-

It is cheaper to use `!= 0` than > `0` for uint256.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
402:> 0
```
#### **Context**:-
[BondingManager.sol#L402](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L402)
```solidity
576:> 0
```
#### **Context**:-
[BondingManager.sol#L576](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L576)
```solidity
606:> 0
```
#### **Context**:-
[BondingManager.sol#L606](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L606)
```solidity
614:> 0
```
#### **Context**:-
[BondingManager.sol#L614](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L614)
```solidity
754:> 0
```
#### **Context**:-
[BondingManager.sol#L754](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L754)
```solidity
871:> 0
```
#### **Context**:-
[BondingManager.sol#L871](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L871)
```solidity
873:> 0
```
#### **Context**:-
[BondingManager.sol#L873](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L873)
```solidity
884:> 0
```
#### **Context**:-
[BondingManager.sol#L884](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L884)
```solidity
1158:> 0
```
#### **Context**:-
[BondingManager.sol#L1158](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1158)
```solidity
1169:> 0
```
#### **Context**:-
[BondingManager.sol#L1169](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1169)
```solidity
1274:> 0
```
#### **Context**:-
[BondingManager.sol#L1274](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1274)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
316:> 0
```
#### **Context**:-
[BondingVotes.sol#L316](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L316)
```solidity
331:> 0
```
#### **Context**:-
[BondingVotes.sol#L331](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L331)
```solidity
436:> 0
```
#### **Context**:-
[BondingVotes.sol#L436](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L436)
```solidity
507:> 0
```
#### **Context**:-
[BondingVotes.sol#L507](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L507)

## **G-12** | Use shift Right/Left instead of division/multiplication if possible

### **Description** :-

A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left. While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
880: / t
```
#### **Context**:-
[BondingManager.sol#L880](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L880)

## **G-13** | Constructors can be marked `payable`

### **Description** :-

Payable functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided. A constructor can safely be marked as payable, since only the deployer would be able to pass funds, and the project itself would not pass any funds.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
141:constructor
```
#### **Context**:-
[BondingManager.sol#L141](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L141)
```solidity
142:constructor
```
#### **Context**:-
[BondingManager.sol#L142](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L142)
```solidity
149:constructor
```
#### **Context**:-
[BondingManager.sol#L149](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L149)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
104:constructor
```
#### **Context**:-
[BondingVotes.sol#L104](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L104)
```solidity
107:constructor
```
#### **Context**:-
[BondingVotes.sol#L107](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L107)
```solidity
File:2023-08-livepeer/contracts/treasury/LivepeerGovernor.sol
```
```solidity
37:constructor
```
#### **Context**:-
[LivepeerGovernor.sol#L37](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L37)
```solidity
38:constructor
```
#### **Context**:-
[LivepeerGovernor.sol#L38](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L38)
```solidity
42:constructor
```
#### **Context**:-
[LivepeerGovernor.sol#L42](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L42)
```solidity
43:constructor
```
#### **Context**:-
[LivepeerGovernor.sol#L43](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L43)
```solidity
File:2023-08-livepeer/contracts/treasury/Treasury.sol
```
```solidity
13:constructor
```
#### **Context**:-
[Treasury.sol#L13](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/Treasury.sol#L13)

## **G-14** | Don't use `SafeMath` once the solidity version is 0.8.0 or greater

### **Description** :-

Version 0.8.0 introduces internal overflow checks, so using SafeMath is redundant and adds overhead

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
17:SafeMath.sol
```
#### **Context**:-
[BondingManager.sol#L17](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L17)
```solidity
File:2023-08-livepeer/contracts/bonding/libraries/EarningsPoolLIP36.sol
```
```solidity
7:SafeMath.sol
```
#### **Context**:-
[EarningsPoolLIP36.sol#L7](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L7)

## **G-15** | Reduce gas usage by moving to Solidity 0.8.19 or later

### **Description** :-

See this [link](https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/#preventing-dead-code-in-runtime-bytecode) for the full details

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
2:pragma solidity 
```
#### **Context**:-
[BondingManager.sol#L2](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L2)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
2:pragma solidity 
```
#### **Context**:-
[BondingVotes.sol#L2](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L2)
```solidity
File:2023-08-livepeer/contracts/bonding/IBondingManager.sol
```
```solidity
2:pragma solidity 
```
#### **Context**:-
[IBondingManager.sol#L2](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L2)
```solidity
File:2023-08-livepeer/contracts/bonding/IBondingVotes.sol
```
```solidity
2:pragma solidity 
```
#### **Context**:-
[IBondingVotes.sol#L2](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingVotes.sol#L2)
```solidity
File:2023-08-livepeer/contracts/bonding/libraries/EarningsPoolLIP36.sol
```
```solidity
2:pragma solidity 
```
#### **Context**:-
[EarningsPoolLIP36.sol#L2](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L2)
```solidity
File:2023-08-livepeer/contracts/bonding/libraries/SortedArrays.sol
```
```solidity
2:pragma solidity 
```
#### **Context**:-
[SortedArrays.sol#L2](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L2)
```solidity
File:2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol
```
```solidity
2:pragma solidity 
```
#### **Context**:-
[GovernorCountingOverridable.sol#L2](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L2)
```solidity
File:2023-08-livepeer/contracts/treasury/IVotes.sol
```
```solidity
2:pragma solidity 
```
#### **Context**:-
[IVotes.sol#L2](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/IVotes.sol#L2)
```solidity
File:2023-08-livepeer/contracts/treasury/LivepeerGovernor.sol
```
```solidity
2:pragma solidity 
```
#### **Context**:-
[LivepeerGovernor.sol#L2](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L2)
```solidity
File:2023-08-livepeer/contracts/treasury/Treasury.sol
```
```solidity
2:pragma solidity 
```
#### **Context**:-
[Treasury.sol#L2](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/Treasury.sol#L2)

## **G-16** | Gas savings can be achieved by changing the model for assigning value to the structure

### **Description** :-

By changing the pattern of assigning value to the structure, gas savings of ~130 per instance are achieved. In addition, this use will provide significant savings in distribution costs.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
706:        newDel.unbondingLocks[newDelUnbondingLockId] = UnbondingLock({ amount: _amount, withdrawRound: withdrawRound });
```
#### **Context**:-
[BondingManager.sol#L706](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L706)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
763:        del.unbondingLocks[unbondingLockId] = UnbondingLock({ amount: _amount, withdrawRound: withdrawRound });
```
#### **Context**:-
[BondingManager.sol#L763](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L763)

## **G-17** | USE BYTES32 INSTEAD OF STRING

### **Description** :-

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
115:(string memory)
```
#### **Context**:-
[BondingVotes.sol#L115](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L115)
```solidity
122:(string memory)
```
#### **Context**:-
[BondingVotes.sol#L122](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L122)
```solidity
145:(string memory)
```
#### **Context**:-
[BondingVotes.sol#L145](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L145)
```solidity
File:2023-08-livepeer/contracts/bonding/IBondingVotes.sol
```
```solidity
20:(string bondingManagerFunction)
```
#### **Context**:-
[IBondingVotes.sol#L20](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingVotes.sol#L20)
```solidity
File:2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol
```
```solidity
76:(string memory)
```
#### **Context**:-
[GovernorCountingOverridable.sol#L76](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L76)
```solidity
File:2023-08-livepeer/contracts/treasury/IVotes.sol
```
```solidity
13:(string memory)
```
#### **Context**:-
[IVotes.sol#L13](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/IVotes.sol#L13)
```solidity
15:(string memory)
```
#### **Context**:-
[IVotes.sol#L15](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/IVotes.sol#L15)

## **G-18** | Part of the code can be pre calculated

### **Description** :-

If You know what data to hash, there is no need to consume more computational power to hash it using keccak256 , you’ll end up consuming 2x amount of gas.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
1652:keccak256("TicketBroker")), "caller must be TicketBroker");
```
#### **Context**:-
[BondingManager.sol#L1652](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1652)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
1656:keccak256("RoundsManager")), "caller must be RoundsManager");
```
#### **Context**:-
[BondingManager.sol#L1656](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1656)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
1660:keccak256("Verifier")), "caller must be Verifier");
```
#### **Context**:-
[BondingManager.sol#L1660](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1660)

## **G-19** |  Multiple `address/ID` mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

### **Description** :-

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
42:mapping(
```
#### **Context**:-
[BondingManager.sol#L42](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L42)
```solidity
67:mapping(
```
#### **Context**:-
[BondingManager.sol#L67](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L67)
```solidity
84:mapping(
```
#### **Context**:-
[BondingManager.sol#L84](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L84)
```solidity
85:mapping(
```
#### **Context**:-
[BondingManager.sol#L85](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L85)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
61:mapping(
```
#### **Context**:-
[BondingVotes.sol#L61](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L61)
```solidity
72:mapping(
```
#### **Context**:-
[BondingVotes.sol#L72](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L72)
```solidity
78:mapping(
```
#### **Context**:-
[BondingVotes.sol#L78](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L78)
```solidity
File:2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol
```
```solidity
52:mapping(
```
#### **Context**:-
[GovernorCountingOverridable.sol#L52](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L52)
```solidity
56:mapping(
```
#### **Context**:-
[GovernorCountingOverridable.sol#L56](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L56)

## **G-20** | Ternary operation is cheaper than if-else statement

### **Description** :-

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-livepeer/contracts/bonding/BondingManager.sol
```
```solidity
432: else {
```
#### **Context**:-
[BondingManager.sol#L432](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L432)
```solidity
438: else {
```
#### **Context**:-
[BondingManager.sol#L438](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L438)
```solidity
564: else {
```
#### **Context**:-
[BondingManager.sol#L564](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L564)
```solidity
965: else {
```
#### **Context**:-
[BondingManager.sol#L965](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L965)
```solidity
1337: else {
```
#### **Context**:-
[BondingManager.sol#L1337](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1337)
```solidity
File:2023-08-livepeer/contracts/bonding/BondingVotes.sol
```
```solidity
345: else {
```
#### **Context**:-
[BondingVotes.sol#L345](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L345)
```solidity
378: else {
```
#### **Context**:-
[BondingVotes.sol#L378](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L378)
```solidity
File:2023-08-livepeer/contracts/bonding/libraries/SortedArrays.sol
```
```solidity
67: else {
```
#### **Context**:-
[SortedArrays.sol#L67](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L67)
```solidity
File:2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol
```
```solidity
157: else {
```
#### **Context**:-
[GovernorCountingOverridable.sol#L157](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L157)
```solidity
205: else {
```
#### **Context**:-
[GovernorCountingOverridable.sol#L205](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L205)