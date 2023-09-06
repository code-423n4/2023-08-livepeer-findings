

### Gas Optimization Issues
|Title|Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
|[G-1] Avoid unnecessary storage updates | [Avoid unnecessary storage updates](#avoid-unnecessary-storage-updates) | 9 | 7200 |
|[G-2] Check Arguments Early | [Check Arguments Early](#check-arguments-early) | 6 | - |
|[G-3] Inefficient Parameter Storage | [Inefficient Parameter Storage](#inefficient-parameter-storage) | 4 | 200 |
|[G-4] Redundant Contract Existence Check in Consecutive External Calls | [Redundant Contract Existence Check in Consecutive External Calls](#redundant-contract-existence-check-in-consecutive-external-calls) | 3 | 300 |
|[G-5] Use assembly to write address storage values | [Use assembly to write address storage values](#use-assembly-to-write-address-storage-values) | 4 | 308 |

Total: 5 issues

#


## Avoid unnecessary storage updates
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 7200

### Description
Avoid updating storage when the value hasn't changed. If the old value is equal to the new value, not re-storing the value will avoid a `SSTORE` operation (costing 2900 gas), potentially at the expense of a `SLOAD` operation (2100 gas) or a `WARMACCESS` operation (100 gas).

<details>

<summary>
There are 9 instances of this issue:

</summary>

###
#
The function `setTreasuryBalanceCeiling()` changes the state variable without first verifying if the values are different.


```
File: contracts/bonding/BondingManager.sol

177    treasuryBalanceCeiling = _ceiling
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L177](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L177)


#
The function `setUnbondingPeriod()` changes the state variable without first verifying if the values are different.


```
File: contracts/bonding/BondingManager.sol

156    unbondingPeriod = _unbondingPeriod
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L156](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L156)


#
The function `updateDelegatorWithEarnings()` changes the state variable without first verifying if the values are different.


```
File: contracts/bonding/BondingManager.sol

1552    del.bondedAmount = currentBondedAmount
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1552](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1552)


#
The function `updateDelegatorWithEarnings()` changes the state variable without first verifying if the values are different.


```
File: contracts/bonding/BondingManager.sol

1536    t.cumulativeRewards = 0
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1536](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1536)


#
The function `updateDelegatorWithEarnings()` changes the state variable without first verifying if the values are different.


```
File: contracts/bonding/BondingManager.sol

1553    del.fees = currentFees
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1553](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1553)


#
The function `updateDelegatorWithEarnings()` changes the state variable without first verifying if the values are different.


```
File: contracts/bonding/BondingManager.sol

1535    t.cumulativeFees = 0
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1535](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1535)


#
The function `updateTranscoderWithFees()` changes the state variable without first verifying if the values are different.


```
File: contracts/bonding/BondingManager.sol

377    t.cumulativeFees = t.cumulativeFees.add(transcoderRewardStakeFees).add(transcoderCommissionFees)
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L377](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L377)


#
The function `updateTranscoderWithRewards()` changes the state variable without first verifying if the values are different.


```
File: contracts/bonding/BondingManager.sol

1481    t.cumulativeRewards = t.cumulativeRewards.add(transcoderRewardStakeRewards).add(transcoderCommissionRewards)
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1481](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1481)


#
The function `updateTranscoderWithRewards()` changes the state variable without first verifying if the values are different.


```
File: contracts/bonding/BondingManager.sol

1470    t.activeCumulativeRewards = t.cumulativeRewards
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1470](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1470)


</details>

# 

## Check Arguments Early
- Severity: Gas Optimization
- Confidence: High

### Description
Checks that require() or revert() statements that check input arguments are at the top of the function. Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a gas load in a function that may ultimately revert in the unhappy case.

<details>

<summary>
There are 6 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/BondingManager.sol

253    require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID")
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L253](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L253)


#

```
File: contracts/bonding/BondingManager.sol

310    require(isRegisteredTranscoder(_transcoder), "transcoder must be registered")
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L310](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L310)


#

```
File: contracts/bonding/BondingManager.sol

563    require(_to != _owner, "INVALID_DELEGATE")
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L563](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L563)


#

```
File: contracts/bonding/BondingManager.sol

582    require(!isRegisteredTranscoder(_owner), "registered transcoders can't delegate towards other addresses")
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L582](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L582)


#

```
File: contracts/bonding/BondingManager.sol

754    require(_amount > 0, "unbond amount must be greater than 0")
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L754](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L754)


#

```
File: contracts/bonding/BondingManager.sol

1573    require(isValidUnbondingLock(_delegator, _unbondingLockId), "invalid unbonding lock ID")
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1573](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1573)


</details>

# 


## Inefficient Parameter Storage
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 200

### Description
When passing function parameters, using the `calldata` area instead of `memory` can improve gas efficiency. Calldata is a read-only area where function arguments and external function calls' parameters are stored.

By using `calldata` for function parameters, you avoid unnecessary gas costs associated with copying data from `calldata` to memory. This is particularly beneficial when the parameter is read-only and doesn't require modification within the contract.

Using `calldata` for function parameters can help optimize gas usage, especially when making external function calls or when the parameter values are provided externally and don't need to be stored persistently within the contract.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/BondingVotes.sol

389    BondingCheckpoint memory previous
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L389](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L389)


#

```
File: contracts/bonding/BondingVotes.sol

390    BondingCheckpoint memory current
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L390](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L390)


#

```
File: contracts/bonding/libraries/EarningsPoolLIP36.sol

20    EarningsPool.Data memory _prevEarningsPool
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L20](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L20)


#

```
File: contracts/bonding/libraries/EarningsPoolLIP36.sol

49    EarningsPool.Data memory _prevEarningsPool
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L49](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L49)


</details>

# 


## Use assembly to write address storage values
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 308

### Description
Using assembly to write address storage values can potentially save gas costs. This detector flags instances where address storage values are written without using assembly.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/BondingManager.sol

608    del.delegateAddress = _to
```
writes to an address storage value without using assembly.

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L608](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L608)


#

```
File: contracts/bonding/BondingManager.sol

771    del.delegateAddress = address(0)
```
writes to an address storage value without using assembly.

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L771](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L771)


#

```
File: contracts/bonding/BondingManager.sol

829    delegators[msg.sender].delegateAddress = _to
```
writes to an address storage value without using assembly.

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L829](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L829)


#

```
File: contracts/bonding/BondingManager.sol

724    newDel.delegateAddress = oldDelDelegate
```
writes to an address storage value without using assembly.

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L724](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L724)


</details>

# 



## Redundant Contract Existence Check in Consecutive External Calls
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 300

### Description
This detector identifies instances where there are consecutive external calls to the same contract, where the subsequent calls could use a low-level call to save gas.

Note: This detector only triggers if the function call does not return any value. 

Prior to 0.8.10, the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent Solidity versions the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low-level calls never check for contract existence. 

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/BondingManager.sol

332    earningsPool.setStake(t.earningsPoolPerRound[lastUpdateRound].totalStake)
```
This call could be replaced with a low-level call because the contract `EarningsPool` has already been checked in <br>`Line: 328    earningsPool.setCommission(t.rewardCut, t.feeShare)`<br>
[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L332](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L332)


#

```
File: contracts/bonding/BondingManager.sol

429    minter().trustedBurnTokens(burnAmount.sub(finderAmount))
```
This call could be replaced with a low-level call because the contract `IMinter` has already been checked in <br>`Line: 426    minter().trustedTransferTokens(_finder, finderAmount)`<br>
[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L429](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L429)


#

```
File: contracts/bonding/BondingManager.sol

868    earningsPool.setStake(t.earningsPoolPerRound[lastUpdateRound].totalStake)
```
This call could be replaced with a low-level call because the contract `EarningsPool` has already been checked in <br>`Line: 860    earningsPool.setCommission(t.rewardCut, t.feeShare)`<br>
[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L868](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L868)


</details>

# 