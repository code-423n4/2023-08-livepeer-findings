
# Gas 

# summary

|      | issue  | instance |
|------|--------|----------|
|[G-01]|internal functions not called by the contract should be removed to save deployment gas|2|
|[G-02]|Use uint256(1)/uint256(2) instead of true/false to save gas for changes|1|
|[G-03]|Using delete instead of setting info struct/array to 0 saves gas|3|
|[G-04]|Uncheck arithmetics operations that can’t underflow/overflow|1|
|[G-05]| Remove the initializer modifier|1|
|[G-06]|Gas savings can be achieved by changing the model for assigning value to the structure (260 gas)|3|
|[G-07]|A modifier used only once and not being inherited should be inlined to save gas|3|
|[G-08]|use Modifiers Instead of Functions To Save Gas|1|
|[G-09]|Use assembly to perform efficient back-to-back calls|4|



## [G-01] internal functions not called by the contract should be removed to save deployment gas


When you define an internal function in a Solidity contract, Solidity generates code for that function and includes it in the contract bytecode. If the function is not called by the contract itself or any of its external functions, then 

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol
64   function __GovernorCountingOverridable_init(uint256 _quota) internal onlyInitializing {

107  function _quorumReached(uint256 _proposalId) internal view virtual override returns (bool) {    
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L64
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L107


## [G‑02] Use uint256(1)/uint256(2) instead of true/false to save gas for changes


```solidity
File: contracts/treasury/GovernorCountingOverridable.sol
148   voter.hasVoted = true;
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L148



## [G-03] Using delete instead of setting info struct/array to 0 saves gas

Using delete instead of setting a struct or array to zero can save gas when clearing the entire data structure. The delete keyword is more gas-efficient because it clears the data structure in a single operation, whereas setting each individual element to zero in a loop can incur additional gas costs.

```solidity
File:  contracts/bonding/BondingManager.sol
773    del.startRound = 0;

1535    t.cumulativeFees = 0;
1536    t.cumulativeRewards = 0;
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L773
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1535
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1536


## [G-04] Uncheck arithmetics operations that can’t underflow/overflow

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

```solidity
188    return _weight - _voter.deductions;
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L188

## [G-05] Remove the initializer modifier

If we can just ensure that the initialize() function could only be called from within the constructor, we shouldn’t need to worry about it getting called again.

```solidity
File:  contracts/treasury/Treasury.sol
21    ) external initializer {
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/Treasury.sol#L21

## [G-06] Gas savings can be achieved by changing the model for assigning value to the structure (260 gas)


By directly assigning the values to the individual fields of the bond struct, you can save gas compared to using the struct initialization syntax. This approach avoids the overhead of creating a new BondingCheckpoint instance and assigning the values using the initialization syntax.

```solidity
File:  contracts/bonding/BondingManager.sol
706    newDel.unbondingLocks[newDelUnbondingLockId] = UnbondingLock({ amount: _amount, withdrawRound: withdrawRound });

763   del.unbondingLocks[unbondingLockId] = UnbondingLock({ amount: _amount, withdrawRound: withdrawRound });
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L706
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L763




```solidity
File: contracts/bonding/BondingVotes.sol
280     BondingCheckpoint memory bond = BondingCheckpoint({
            bondedAmount: _bondedAmount,
            delegateAddress: _delegateAddress,
            delegatedAmount: _delegatedAmount,
            lastClaimRound: _lastClaimRound,
            lastRewardRound: _lastRewardRound
        });
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L280



## [G-07] A modifier used only once and not being inherited should be inlined to save gas



By inlining a modifier that is used only once and not being inherited, you can eliminate the overhead of the generated code and reduce the gas cost of your contract.

```solidity
File:  contracts/bonding/BondingManager.sol
106     modifier onlyTicketBroker() {

112     modifier onlyRoundsManager() {    

118     modifier onlyVerifier() {    
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L106
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L112
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L118




## [G-08] Use Modifiers Instead of Functions To Save Gas

This with 0.8.9 compiler and optimization enabled. As you can see it's cheaper to deploy with a modifier, and it will save you about 30 gas. But sometimes modifiers increase code size of the contract.

```solidity
File:  contracts/bonding/BondingVotes.sol
553 function _onlyBondingManager() internal view {
        if (msg.sender != address(bondingManager())) {
            revert InvalidCaller(msg.sender, address(bondingManager()));
        }
    }
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L553-L557



## [G-09] Use assembly to perform efficient back-to-back calls

If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.


```solidity
1323     if (transcoderPool.contains(_delegate)) {
1324     transcoderPool.updateKey(_delegate, newStake, _newPosPrev, _newPosNext);
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1323-L1323
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1323-L1324

```solidity
1401     if (transcoderPool.isFull()) {
1402     address lastTranscoder = transcoderPool.getLast();
```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1401-L1401
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1401-L1402



