### Medium Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[M-1] Fee-on-transfer detection | [Fee-on-transfer detection](#fee-on-transfer-detection) | 1 |
|[M-2] Calls transfer()/transferFrom() With IERC20 | [Calls transfer()/transferFrom() With IERC20](#calls-transfertransferfrom-with-ierc20) | 1 |

Total: 2 issues


### Low Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[L-1] Missing gap Storage Variable in Upgradeable Contract | [Missing gap Storage Variable in Upgradeable Contract](#missing-gap-storage-variable-in-upgradeable-contract) | 3 |
|[L-2] Upgradeable contract uses non-upgradeable version of the OpenZeppelin libraries/contracts | [Upgradeable contract uses non-upgradeable version of the OpenZeppelin libraries/contracts](#upgradeable-contract-uses-non-upgradeable-version-of-the-openzeppelin-librariescontracts) | 1 |
|[L-3] Reentrancy vulnerabilities | [Reentrancy vulnerabilities](#reentrancy-vulnerabilities) | 6 |
|[L-4] Restrictive casts in assignments | [Restrictive casts in assignments](#restrictive-casts-in-assignments) | 1 |
|[L-5] Setter No Initial Value Check Detection | [Setter No Initial Value Check Detection](#setter-no-initial-value-check-detection) | 3 |
|[L-6] Arrays can grow without a way to shrink them | [Arrays can grow without a way to shrink them](#arrays-can-grow-without-a-way-to-shrink-them) | 1 |

Total: 6 issues


### Non-Critical Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[N-``] Do not calculate constants | [Do not calculate constants](#do-not-calculate-constants) | 1 |
|[N-1] Inconsistent usage of require/error | [Inconsistent usage of require/error](#inconsistent-usage-of-requireerror) | 1 |
|[N-3] Event Emission Preceding External Calls: A Best Practice | [Event Emission Preceding External Calls: A Best Practice](#event-emission-preceding-external-calls-a-best-practice) | 16 |
|[N-4] Should Declare as Pure | [Should Declare as Pure](#should-declare-as-pure) | 6 |
|[N-5] Should Declare as View | [Should Declare as View](#should-declare-as-view) | 1 |
|[N-6] Variable names too similar | [Variable names too similar](#variable-names-too-similar) | 21 |
|[N-7] Functions that alter state should emit events | [Functions that alter state should emit events](#functions-that-alter-state-should-emit-events) | 3 |
|[N-8] Unused imports | [Unused imports](#unused-imports) | 4 |
|[N-9] Whitespace in Expressions | [Whitespace in Expressions](#whitespace-in-expressions) | 2 |

Total: 9 issues

## Fee-on-transfer detection
- Severity: Medium
- Confidence: Medium

### Description
Contracts may be vulnerable to errors due to fee-on-transfer tokens. Specifically, some functions transfer tokens without confirming that the actual received amount aligns with the originally intended transfer amount. A discrepancy may occur if the token has a transfer fee, resulting in the final balance being less than anticipated. Moreover, an attacker could potentially exploit latent funds for an unwarranted credit. To prevent these issues, balances should be accurately tracked before and after transfers, using the difference in balance as the actual transferred amount instead of the initial intended amount. 

Example: If a contract sends 10 tokens, but a 1% fee is applied on transfer, the recipient only receives 9.9 tokens. If the contract does not account for this fee, it may inaccurately record that 10 tokens were sent, leading to accounting discrepancies.

NOTE: This detector conducts a recursive check to confirm that 'balanceOf' is properly checked both before and after any token transfer operations, to guarantee accurate accounting.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#
<br><br>
***
```
File: contracts/bonding/BondingManager.sol

616    livepeerToken().transferFrom(msg.sender, address(minter()), _amount)
```
No balance check detected before the transfer. The specified transfer amount: `_amount` might be inaccurate due to possible fee-on-transfer accounting discrepancies. <br><br>use here ->   
```
File: contracts/bonding/BondingManager.sol

552    uint256 delegationAmount = _amount
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L616](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L616)


</details>

# 


## Calls transfer()/transferFrom() With IERC20
- Severity: Medium
- Confidence: High

### Description
The transfer() and transferFrom() methods are part of the ERC-20 standard, designed for transferring tokens between accounts. However, these methods could be risky when transferring tokens to smart contracts because they don't guarantee that the receiving smart contract will handle the transferred tokens correctly, potentially leading to permanent token loss. ERC-20 standard doesn't include a 'onTokenReceived' function or similar to alert the recipient about incoming transfers. Therefore, the 'transfer' and 'transferFrom' functions are not reliable for interactions with arbitrary contracts. A safer alternative is to use safeTransfer() and safeTransferFrom() methods provided by libraries like OpenZeppelin's SafeERC20. These methods add a layer of protection by checking if the recipient is a contract and if it has a function to handle incoming tokens. If these checks are not passed, the transfer operation is not executed, hence preventing potential token loss.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/BondingManager.sol

616    livepeerToken().transferFrom(msg.sender, address(minter()), _amount)
```
 use safeTransferFrom instead.

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L616](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L616)


</details>

# 


### Low Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[L-1] Require Should Used Instead of Assert | [Require Should Used Instead of Assert](#require-should-used-instead-of-assert) | 1 |
|[L-2] Use of Solidity version 0.8.20 may cause incompatibility issues | [Use of Solidity version 0.8.20 may cause incompatibility issues](#use-of-solidity-version-0820-may-cause-incompatibility-issues) | 1 |
|[L-3] Missing gap Storage Variable in Upgradeable Contract | [Missing gap Storage Variable in Upgradeable Contract](#missing-gap-storage-variable-in-upgradeable-contract) | 3 |
|[L-4] Upgradeable contract uses non-upgradeable version of the OpenZeppelin libraries/contracts | [Upgradeable contract uses non-upgradeable version of the OpenZeppelin libraries/contracts](#upgradeable-contract-uses-non-upgradeable-version-of-the-openzeppelin-librariescontracts) | 4 |
|[L-5] Reentrancy vulnerabilities | [Reentrancy vulnerabilities](#reentrancy-vulnerabilities) | 6 |
|[L-6] Restrictive casts in assignments | [Restrictive casts in assignments](#restrictive-casts-in-assignments) | 1 |
|[L-7] Setter No Initial Value Check Detection | [Setter No Initial Value Check Detection](#setter-no-initial-value-check-detection) | 3 |
|[L-8] Arrays can grow without a way to shrink them | [Arrays can grow without a way to shrink them](#arrays-can-grow-without-a-way-to-shrink-them) | 1 |

Total: 20 instances over 8 issues

## Missing gap Storage Variable in Upgradeable Contract
- Severity: Low
- Confidence: High

### Note
I reported only on instances that were mising in the wining bot. 

### Description
upgradeable contracts that are missing a '__gap' storage variable. In upgradeable contracts, it is important to reserve storage slots for future versions to introduce new storage variables. The '__gap' storage variable acts as a placeholder, allowing for seamless upgrades without affecting existing storage layout.When a contract is not designed with a '__gap' storage variable, adding new storage variables in subsequent versions becomes problematic. It can lead to storage collisions or layout incompatibilities, making it difficult to upgrade the contract without requiring costly data migrations or redeployments.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/BondingVotes.sol

20    contract BondingVotes is ManagerProxyTarget, IBondingVotes 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L20-L558](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L20-L558)


#

```
File: contracts/bonding/IBondingVotes.sol

9    interface IBondingVotes is IVotes 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingVotes.sol#L9-L53](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingVotes.sol#L9-L53)


#

```
File: contracts/treasury/IVotes.sol

6    interface IVotes is IERC5805Upgradeable 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/IVotes.sol#L6-L18](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/IVotes.sol#L6-L18)


</details>

# 



## Upgradeable contract uses non-upgradeable version of the OpenZeppelin libraries/contracts
- Severity: Low
- Confidence: High

## Note 
I reported only on instances that were mising in the wining bot. 

### Description
Detects when upgradeable contracts use non-upgradeable OpenZeppelin libraries/contracts.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###


```
File: contracts/bonding/BondingVotes.sol

5    import "@openzeppelin/contracts/utils/math/SafeCast.sol";
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L5](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L5)


</details>

# 


## Reentrancy vulnerabilities
- Severity: Low
- Confidence: Medium

### Description

Detection of the [reentrancy bug](https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy).
Only report reentrancy that acts as a double call (see `reentrancy-eth`, `reentrancy-no-eth`).

<details>

<summary>
There are 6 instances of this issue:

</summary>

###
#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

1352    function decreaseTotalStake(

1353            address _delegate,

1354            uint256 _amount,

1355            address _newPosPrev,

1356            address _newPosNext

1357        ) internal autoCheckpoint(_delegate) 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

1357    autoCheckpoint(_delegate)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	State variables written after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

1368    nextRoundTotalActiveStake = nextRoundTotalActiveStake.sub(_amount)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1352-L1384](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1352-L1384)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

1294    function increaseTotalStake(

1295            address _delegate,

1296            uint256 _amount,

1297            address _newPosPrev,

1298            address _newPosNext

1299        ) internal autoCheckpoint(_delegate) 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

1299    autoCheckpoint(_delegate)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	State variables written after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

1300    return increaseTotalStakeUncheckpointed(_delegate, _amount, _newPosPrev, _newPosNext)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1325    nextRoundTotalActiveStake = nextRoundTotalActiveStake.add(_amount)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1430    nextRoundTotalActiveStake = pendingNextRoundTotalActiveStake
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1294-L1301](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1294-L1301)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

842    function rewardWithHint(address _newPosPrev, address _newPosNext)

843            public

844            whenSystemNotPaused

845            currentRoundInitialized

846            autoCheckpoint(msg.sender)

847        
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

846    autoCheckpoint(msg.sender)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	State variables written after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

875    _setTreasuryRewardCutRate(0)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1179    nextRoundTreasuryRewardCutRate = _cutRate
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L842-L900](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L842-L900)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

842    function rewardWithHint(address _newPosPrev, address _newPosNext)

843            public

844            whenSystemNotPaused

845            currentRoundInitialized

846            autoCheckpoint(msg.sender)

847        
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

882    uint256 totalRewardTokens = mtr.createReward(earningsPool.totalStake, currentRoundTotalActiveStake)
```

	- 
```
File: contracts/bonding/BondingManager.sol

887    mtr.trustedTransferTokens(trsry, treasuryRewards)
```

	- 
```
File: contracts/bonding/BondingManager.sol

846    autoCheckpoint(msg.sender)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	State variables written after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

894    updateTranscoderWithRewards(msg.sender, transcoderRewards, currentRound, _newPosPrev, _newPosNext)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1325    nextRoundTotalActiveStake = nextRoundTotalActiveStake.add(_amount)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1430    nextRoundTotalActiveStake = pendingNextRoundTotalActiveStake
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L842-L900](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L842-L900)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

394    function slashTranscoder(

395            address _transcoder,

396            address _finder,

397            uint256 _slashAmount,

398            uint256 _finderFee

399        ) external whenSystemNotPaused onlyVerifier autoClaimEarnings(_transcoder) autoCheckpoint(_transcoder) 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

399    autoCheckpoint(_transcoder)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	State variables written after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

407    resignTranscoder(_transcoder)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1443    nextRoundTotalActiveStake = nextRoundTotalActiveStake.sub(transcoderTotalStake(_transcoder))
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L394-L441](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L394-L441)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

745    function unbondWithHint(

746            uint256 _amount,

747            address _newPosPrev,

748            address _newPosNext

749        ) public whenSystemNotPaused currentRoundInitialized autoClaimEarnings(msg.sender) autoCheckpoint(msg.sender) 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

749    autoCheckpoint(msg.sender)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	State variables written after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

776    resignTranscoder(msg.sender)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1443    nextRoundTotalActiveStake = nextRoundTotalActiveStake.sub(transcoderTotalStake(_transcoder))
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L745-L784](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L745-L784)


</details>

# 


## Restrictive casts in assignments
- Severity: Low
- Confidence: Medium

### Description
This detector flags the instances where a restrictive cast is being used in an assignment. For example, if an address variable 'foo' is used in an expression like 'IERC20 token = FooToken(foo)', the specific cast to 'FooToken' is unnecessary as the compiler only checks that 'FooToken' extends 'IERC20' and does not check any function signatures. In such cases, it would be more appropriate to use 'IERC20 token = IERC20(foo)' or even better, 'FooToken token = FooToken(foo)'. The former allows the file in which it's used to possibly remove the import for 'FooToken', leading to cleaner and more efficient code.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/BondingVotes.sol

96    uint256 currentRound = clock()
```

```
//  @audit: uint256 vs uint48 
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L96](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L96)


</details>

# 


## Setter No Initial Value Check Detection
- Severity: Low
- Confidence: Medium

### Description
This detector flags setter functions that lack an initial value check. Lack of such a check can lead to assigning wrong values to the variable, causing unexpected contract behavior.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/BondingManager.sol

155    function setUnbondingPeriod(uint64 _unbondingPeriod) external onlyControllerOwner 
```
 Setter lacks an initial value check
[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L155-L159](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L155-L159)


#

```
File: contracts/bonding/BondingManager.sol

176    function setTreasuryBalanceCeiling(uint256 _ceiling) external onlyControllerOwner 
```
 Setter lacks an initial value check
[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L176-L180](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L176-L180)


#

```
File: contracts/bonding/BondingManager.sol

1176    function _setTreasuryRewardCutRate(uint256 _cutRate) internal 
```
 Setter lacks an initial value check
[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1176-L1182](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1176-L1182)


</details>

# 


## Arrays can grow without a way to shrink them
- Severity: Low
- Confidence: High

### Description
It's a good practice to maintain control over the size of array state variables in Solidity, especially if they are dynamically updated. If a contract includes a mechanism to push items into an array, it should ideally also provide a mechanism to remove items. Ignoring this can lead to bloated and inefficient contracts.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/libraries/SortedArrays.sol

64    uint256[] storage array
```
Array grow:<br>
```
File: contracts/bonding/libraries/SortedArrays.sol

66    array.push(val)
```

```
File: contracts/bonding/libraries/SortedArrays.sol

77    array.push(val)
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L64](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L64)


</details>

# 



## Do not calculate constants
- Severity: Non-Critical
- Confidence: High

### Description
Due to how constant variables are implemented in Solidity (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas. While in most cases, the compiler will optimize these computations away, it is considered a best practice to write code that does not rely on the compiler optimization.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/BondingManager.sol

32    uint256 constant MAX_FUTURE_ROUND = 2**256 - 1
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L32](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L32)


</details>

# 


## Inconsistent usage of require/error
- Severity: Non-Critical
- Confidence: High

### Description
Some parts of the codebase use require statements, while others use custom errors. Consider refactoring the code to use the same approach: the following findings represent the minority of require vs error, and they show one occurrence in each contract, for brevity.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/libraries/SortedArrays.sol

12    library SortedArrays 
```

```
File: contracts/bonding/libraries/SortedArrays.sol

72    revert DecreasingValues(val, last)
```
The contract `SortedArrays` uses both `require` and custom `revert` for error handling.

There are 1 `require` statement(s) and 1 custom `revert` statement(s).

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L12-L81](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L12-L81)


</details>

# 



## Event Emission Preceding External Calls: A Best Practice
- Severity: Non-Critical
- Confidence: Medium

### Description

Ensure that events follow the best practice of check-effects-interaction, and are emitted before external calls


<details>

<summary>
There are 16 instances of this issue:

</summary>

###
#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

537    function bondForWithHint(

538            uint256 _amount,

539            address _owner,

540            address _to,

541            address _oldDelegateNewPosPrev,

542            address _oldDelegateNewPosNext,

543            address _currDelegateNewPosPrev,

544            address _currDelegateNewPosNext

545        ) public whenSystemNotPaused currentRoundInitialized 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

589    decreaseTotalStake(currentDelegate, currentBondedAmount, _oldDelegateNewPosPrev, _oldDelegateNewPosNext)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	- 
```
File: contracts/bonding/BondingManager.sol

612    increaseTotalStake(_to, delegationAmount, _currDelegateNewPosPrev, _currDelegateNewPosNext)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

1431    emit TranscoderActivated(_transcoder, _activationRound)
```

		- 
```
File: contracts/bonding/BondingManager.sol

612    increaseTotalStake(_to, delegationAmount, _currDelegateNewPosPrev, _currDelegateNewPosNext)
```

	- 
```
File: contracts/bonding/BondingManager.sol

1420    emit TranscoderDeactivated(lastTranscoder, _activationRound)
```

		- 
```
File: contracts/bonding/BondingManager.sol

612    increaseTotalStake(_to, delegationAmount, _currDelegateNewPosPrev, _currDelegateNewPosNext)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L537-L623](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L537-L623)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

537    function bondForWithHint(

538            uint256 _amount,

539            address _owner,

540            address _to,

541            address _oldDelegateNewPosPrev,

542            address _oldDelegateNewPosNext,

543            address _currDelegateNewPosPrev,

544            address _currDelegateNewPosNext

545        ) public whenSystemNotPaused currentRoundInitialized 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

589    decreaseTotalStake(currentDelegate, currentBondedAmount, _oldDelegateNewPosPrev, _oldDelegateNewPosNext)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	- 
```
File: contracts/bonding/BondingManager.sol

612    increaseTotalStake(_to, delegationAmount, _currDelegateNewPosPrev, _currDelegateNewPosNext)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	- 
```
File: contracts/bonding/BondingManager.sol

616    livepeerToken().transferFrom(msg.sender, address(minter()), _amount)
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

619    emit Bond(_to, currentDelegate, _owner, _amount, del.bondedAmount)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L537-L623](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L537-L623)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

447    function claimEarnings(uint256 _endRound)

448            external

449            whenSystemNotPaused

450            currentRoundInitialized

451            autoCheckpoint(msg.sender)

452        
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

451    autoCheckpoint(msg.sender)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

1541    emit EarningsClaimed(

1542                del.delegateAddress,

1543                _delegator,

1544                currentBondedAmount.sub(del.bondedAmount),

1545                currentFees.sub(del.fees),

1546                startRound,

1547                _endRound

1548            )
```

		- 
```
File: contracts/bonding/BondingManager.sol

456    _autoClaimEarnings(msg.sender)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L447-L457](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L447-L457)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

1294    function increaseTotalStake(

1295            address _delegate,

1296            uint256 _amount,

1297            address _newPosPrev,

1298            address _newPosNext

1299        ) internal autoCheckpoint(_delegate) 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

1299    autoCheckpoint(_delegate)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

1431    emit TranscoderActivated(_transcoder, _activationRound)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1300    return increaseTotalStakeUncheckpointed(_delegate, _amount, _newPosPrev, _newPosNext)
```

	- 
```
File: contracts/bonding/BondingManager.sol

1420    emit TranscoderDeactivated(lastTranscoder, _activationRound)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1300    return increaseTotalStakeUncheckpointed(_delegate, _amount, _newPosPrev, _newPosNext)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1294-L1301](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1294-L1301)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

1564    function processRebond(

1565            address _delegator,

1566            uint256 _unbondingLockId,

1567            address _newPosPrev,

1568            address _newPosNext

1569        ) internal autoCheckpoint(_delegator) 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

1582    increaseTotalStake(del.delegateAddress, amount, _newPosPrev, _newPosNext)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	- 
```
File: contracts/bonding/BondingManager.sol

1569    autoCheckpoint(_delegator)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

1584    emit Rebond(del.delegateAddress, _delegator, _unbondingLockId, amount)
```

	- 
```
File: contracts/bonding/BondingManager.sol

1431    emit TranscoderActivated(_transcoder, _activationRound)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1582    increaseTotalStake(del.delegateAddress, amount, _newPosPrev, _newPosNext)
```

	- 
```
File: contracts/bonding/BondingManager.sol

1420    emit TranscoderDeactivated(lastTranscoder, _activationRound)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1582    increaseTotalStake(del.delegateAddress, amount, _newPosPrev, _newPosNext)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1564-L1585](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1564-L1585)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

842    function rewardWithHint(address _newPosPrev, address _newPosNext)

843            public

844            whenSystemNotPaused

845            currentRoundInitialized

846            autoCheckpoint(msg.sender)

847        
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

846    autoCheckpoint(msg.sender)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

1181    emit ParameterUpdate("nextRoundTreasuryRewardCutRate")
```

		- 
```
File: contracts/bonding/BondingManager.sol

875    _setTreasuryRewardCutRate(0)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L842-L900](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L842-L900)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

842    function rewardWithHint(address _newPosPrev, address _newPosNext)

843            public

844            whenSystemNotPaused

845            currentRoundInitialized

846            autoCheckpoint(msg.sender)

847        
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

882    uint256 totalRewardTokens = mtr.createReward(earningsPool.totalStake, currentRoundTotalActiveStake)
```

	- 
```
File: contracts/bonding/BondingManager.sol

887    mtr.trustedTransferTokens(trsry, treasuryRewards)
```

	- 
```
File: contracts/bonding/BondingManager.sol

846    autoCheckpoint(msg.sender)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

899    emit Reward(msg.sender, transcoderRewards)
```

	- 
```
File: contracts/bonding/BondingManager.sol

1431    emit TranscoderActivated(_transcoder, _activationRound)
```

		- 
```
File: contracts/bonding/BondingManager.sol

894    updateTranscoderWithRewards(msg.sender, transcoderRewards, currentRound, _newPosPrev, _newPosNext)
```

	- 
```
File: contracts/bonding/BondingManager.sol

1420    emit TranscoderDeactivated(lastTranscoder, _activationRound)
```

		- 
```
File: contracts/bonding/BondingManager.sol

894    updateTranscoderWithRewards(msg.sender, transcoderRewards, currentRound, _newPosPrev, _newPosNext)
```

	- 
```
File: contracts/bonding/BondingManager.sol

889    emit TreasuryReward(msg.sender, trsry, treasuryRewards)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L842-L900](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L842-L900)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

394    function slashTranscoder(

395            address _transcoder,

396            address _finder,

397            uint256 _slashAmount,

398            uint256 _finderFee

399        ) external whenSystemNotPaused onlyVerifier autoClaimEarnings(_transcoder) autoCheckpoint(_transcoder) 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

399    autoCheckpoint(_transcoder)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

1446    emit TranscoderDeactivated(_transcoder, deactivationRound)
```

		- 
```
File: contracts/bonding/BondingManager.sol

407    resignTranscoder(_transcoder)
```

	- 
```
File: contracts/bonding/BondingManager.sol

439    emit TranscoderSlashed(_transcoder, _finder, 0, 0)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L394-L441](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L394-L441)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

394    function slashTranscoder(

395            address _transcoder,

396            address _finder,

397            uint256 _slashAmount,

398            uint256 _finderFee

399        ) external whenSystemNotPaused onlyVerifier autoClaimEarnings(_transcoder) autoCheckpoint(_transcoder) 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

426    minter().trustedTransferTokens(_finder, finderAmount)
```

	- 
```
File: contracts/bonding/BondingManager.sol

429    minter().trustedBurnTokens(burnAmount.sub(finderAmount))
```

	- 
```
File: contracts/bonding/BondingManager.sol

399    autoCheckpoint(_transcoder)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

431    emit TranscoderSlashed(_transcoder, _finder, penalty, finderAmount)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L394-L441](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L394-L441)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

394    function slashTranscoder(

395            address _transcoder,

396            address _finder,

397            uint256 _slashAmount,

398            uint256 _finderFee

399        ) external whenSystemNotPaused onlyVerifier autoClaimEarnings(_transcoder) autoCheckpoint(_transcoder) 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

434    minter().trustedBurnTokens(burnAmount)
```

	- 
```
File: contracts/bonding/BondingManager.sol

399    autoCheckpoint(_transcoder)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

436    emit TranscoderSlashed(_transcoder, address(0), penalty, 0)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L394-L441](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L394-L441)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

679    function transferBond(

680            address _delegator,

681            uint256 _amount,

682            address _oldDelegateNewPosPrev,

683            address _oldDelegateNewPosNext,

684            address _newDelegateNewPosPrev,

685            address _newDelegateNewPosNext

686        ) public whenSystemNotPaused currentRoundInitialized 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

695    unbondWithHint(_amount, _oldDelegateNewPosPrev, _oldDelegateNewPosNext)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

1541    emit EarningsClaimed(

1542                del.delegateAddress,

1543                _delegator,

1544                currentBondedAmount.sub(del.bondedAmount),

1545                currentFees.sub(del.fees),

1546                startRound,

1547                _endRound

1548            )
```

		- 
```
File: contracts/bonding/BondingManager.sol

715    updateDelegatorWithEarnings(_delegator, currentRound, lastClaimRound)
```

	- 
```
File: contracts/bonding/BondingManager.sol

709    emit TransferBond(msg.sender, _delegator, oldDelUnbondingLockId, newDelUnbondingLockId, _amount)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L679-L734](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L679-L734)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

679    function transferBond(

680            address _delegator,

681            uint256 _amount,

682            address _oldDelegateNewPosPrev,

683            address _oldDelegateNewPosNext,

684            address _newDelegateNewPosPrev,

685            address _newDelegateNewPosNext

686        ) public whenSystemNotPaused currentRoundInitialized 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

695    unbondWithHint(_amount, _oldDelegateNewPosPrev, _oldDelegateNewPosNext)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	- 
```
File: contracts/bonding/BondingManager.sol

733    processRebond(_delegator, newDelUnbondingLockId, _newDelegateNewPosPrev, _newDelegateNewPosNext)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

1584    emit Rebond(del.delegateAddress, _delegator, _unbondingLockId, amount)
```

		- 
```
File: contracts/bonding/BondingManager.sol

733    processRebond(_delegator, newDelUnbondingLockId, _newDelegateNewPosPrev, _newDelegateNewPosNext)
```

	- 
```
File: contracts/bonding/BondingManager.sol

1431    emit TranscoderActivated(_transcoder, _activationRound)
```

		- 
```
File: contracts/bonding/BondingManager.sol

733    processRebond(_delegator, newDelUnbondingLockId, _newDelegateNewPosPrev, _newDelegateNewPosNext)
```

	- 
```
File: contracts/bonding/BondingManager.sol

1420    emit TranscoderDeactivated(lastTranscoder, _activationRound)
```

		- 
```
File: contracts/bonding/BondingManager.sol

733    processRebond(_delegator, newDelUnbondingLockId, _newDelegateNewPosPrev, _newDelegateNewPosNext)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L679-L734](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L679-L734)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

745    function unbondWithHint(

746            uint256 _amount,

747            address _newPosPrev,

748            address _newPosNext

749        ) public whenSystemNotPaused currentRoundInitialized autoClaimEarnings(msg.sender) autoCheckpoint(msg.sender) 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

749    autoCheckpoint(msg.sender)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

1446    emit TranscoderDeactivated(_transcoder, deactivationRound)
```

		- 
```
File: contracts/bonding/BondingManager.sol

776    resignTranscoder(msg.sender)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L745-L784](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L745-L784)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

745    function unbondWithHint(

746            uint256 _amount,

747            address _newPosPrev,

748            address _newPosNext

749        ) public whenSystemNotPaused currentRoundInitialized autoClaimEarnings(msg.sender) autoCheckpoint(msg.sender) 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

781    decreaseTotalStake(currentDelegate, _amount, _newPosPrev, _newPosNext)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	- 
```
File: contracts/bonding/BondingManager.sol

749    autoCheckpoint(msg.sender)
```

		- 
```
File: contracts/bonding/BondingManager.sol

1600    bondingVotes().checkpointBondingState(

1601                _owner,

1602                startRound,

1603                _delegator.bondedAmount,

1604                _delegator.delegateAddress,

1605                _delegator.delegatedAmount,

1606                _delegator.lastClaimRound,

1607                _transcoder.lastRewardRound

1608            )
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

783    emit Unbond(currentDelegate, msg.sender, unbondingLockId, _amount, withdrawRound)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L745-L784](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L745-L784)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

273    function withdrawFees(address payable _recipient, uint256 _amount)

274            external

275            whenSystemNotPaused

276            currentRoundInitialized

277            autoClaimEarnings(msg.sender)

278        
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

285    minter().trustedWithdrawETH(_recipient, _amount)
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

287    emit WithdrawFees(msg.sender, _recipient, _amount)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L273-L288](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L273-L288)


#
Reentrancy in 
```
File: contracts/bonding/BondingManager.sol

249    function withdrawStake(uint256 _unbondingLockId) external whenSystemNotPaused currentRoundInitialized 
```
:
	External calls:
	- 
```
File: contracts/bonding/BondingManager.sol

265    minter().trustedTransferTokens(msg.sender, amount)
```

	Event emitted after the call(s):
	- 
```
File: contracts/bonding/BondingManager.sol

267    emit WithdrawStake(msg.sender, _unbondingLockId, amount, withdrawRound)
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L249-L268](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L249-L268)


</details>

# 


## Should Declare as Pure
- Severity: Non-Critical
- Confidence: High

### Description
Detects when functions do not change the state of the contract and can be declared as pure.

<details>

<summary>
There are 6 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/BondingManager.sol

1189    function cumulativeFactorsPool(Transcoder storage _transcoder, uint256 _round)

1190            internal

1191            view

1192            returns (EarningsPool.Data memory pool)

1193        
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1189-L1198](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1189-L1198)


#

```
File: contracts/bonding/BondingVotes.sol

387    function onBondingCheckpointChanged(

388            address _account,

389            BondingCheckpoint memory previous,

390            BondingCheckpoint memory current

391        ) internal 
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L387-L412](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L387-L412)


#

```
File: contracts/bonding/libraries/EarningsPoolLIP36.sol

18    function updateCumulativeFeeFactor(

19            EarningsPool.Data storage earningsPool,

20            EarningsPool.Data memory _prevEarningsPool,

21            uint256 _fees

22        ) internal 
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L18-L39](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L18-L39)


#

```
File: contracts/bonding/libraries/EarningsPoolLIP36.sol

47    function updateCumulativeRewardFactor(

48            EarningsPool.Data storage earningsPool,

49            EarningsPool.Data memory _prevEarningsPool,

50            uint256 _rewards

51        ) internal 
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L47-L59](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L47-L59)


#

```
File: contracts/bonding/libraries/SortedArrays.sol

28    function findLowerBound(uint256[] storage _array, uint256 _val) internal view returns (uint256) 
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L28-L55](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L28-L55)


#

```
File: contracts/bonding/libraries/SortedArrays.sol

64    function pushSorted(uint256[] storage array, uint256 val) internal 
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L64-L80](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L64-L80)


</details>

# 


## Should Declare as View
- Severity: Non-Critical
- Confidence: High

### Description
Detects when functions do not change the state of the contract and can be declared as view.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/BondingManager.sol

1591    function _checkpointBondingState(

1592            address _owner,

1593            Delegator storage _delegator,

1594            Transcoder storage _transcoder

1595        ) internal 
```

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1591-L1609](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1591-L1609)


</details>

# 


## Variable names too similar
- Severity: Non-Critical
- Confidence: Medium

### Description
Detect variables with names that are too similar.

<details>

<summary>
There are 21 instances of this issue:

</summary>

###
#
Variable 
```
File: contracts/bonding/BondingManager.sol

1460    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1460](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1460)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

1393    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1393](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1393)


#
Variable 
```
File: contracts/bonding/IBondingManager.sol

66    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L66](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L66)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

1206    Transcoder storage _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1206](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1206)


#
Variable 
```
File: contracts/bonding/IBondingManager.sol

79    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L79](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L79)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

937    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L937](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L937)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

1128    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1128](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1128)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

1027    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1027](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1027)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

303    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L303](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L303)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

946    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L946](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L946)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

1189    Transcoder storage _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1189](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1189)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

1239    Transcoder storage _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1239](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1239)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

1145    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1145](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1145)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

1594    Transcoder storage _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1594](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1594)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

395    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L395](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L395)


#
Variable 
```
File: contracts/bonding/IBondingManager.sol

85    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L85](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L85)


#
Variable 
```
File: contracts/bonding/IBondingManager.sol

60    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L60](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L60)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

987    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L987](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L987)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

1156    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1156](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1156)


#
Variable 
```
File: contracts/bonding/BondingManager.sol

1437    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1437](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1437)


#
Variable 
```
File: contracts/bonding/IBondingManager.sol

77    address _transcoder
```
 is too similar to 
```
File: contracts/bonding/BondingManager.sol

85    mapping(address => Transcoder) private transcoders
```


[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L77](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L77)


</details>

# 


## Functions that alter state should emit events
- Severity: Non-Critical
- Confidence: Medium

### Description
Functions that alter the state of the contract should emit an event to inform external observers of the change.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/BondingManager.sol

302    function updateTranscoderWithFees(

303            address _transcoder,

304            uint256 _fees,

305            uint256 _round

306        ) external whenSystemNotPaused onlyTicketBroker 
```
 The function `updateTranscoderWithFees` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L302-L385](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L302-L385)


#

```
File: contracts/bonding/BondingManager.sol

1352    function decreaseTotalStake(

1353            address _delegate,

1354            uint256 _amount,

1355            address _newPosPrev,

1356            address _newPosNext

1357        ) internal autoCheckpoint(_delegate) 
```
 The function `decreaseTotalStake` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1352-L1384](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1352-L1384)


#

```
File: contracts/bonding/BondingVotes.sol

303    function checkpointTotalActiveStake(uint256 _totalStake, uint256 _round) external virtual onlyBondingManager 
```
 The function `checkpointTotalActiveStake` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L303-L310](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L303-L310)


</details>

# 


## Whitespace in Expressions
- Severity: Non-Critical
- Confidence: High

### Description
Detects when whitespace usage in expressions does not conform to the Solidity style guide.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: contracts/bonding/BondingManager.sol

23    contract BondingManager is ManagerProxyTarget, IBondingManager 
```

```
// @audit: whitespace inside parenthesis
Line: 747            (uint256 stake, ) = pendingStakeAndFees(_delegator, endRound);


```
[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L23-L1674](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L23-L1674)


#

```
File: contracts/bonding/BondingVotes.sol

20    contract BondingVotes is ManagerProxyTarget, IBondingVotes 
```

```
// @audit: whitespace inside parenthesis
Line: 103            (uint256 amount, ) = getBondingStateAt(_account, clock() + 1);


// @audit: whitespace inside parenthesis
Line: 109            (uint256 amount, ) = getBondingStateAt(_account, _round + 1);


// @audit: whitespace inside parenthesis
Line: 341            (uint256 stakeWithRewards, ) = EarningsPoolLIP36.delegatorCumulativeStakeAndFees(


```
[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L20-L558](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L20-L558)


</details>

# 


## Unused imports
- Severity: Non-Critical
- Confidence: High

### Description
Identify unused imports in source files that can be safely removed. Please note that this detector does not support files with cyclic imports or files that use 'import {...} from' directives.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
#
Consider removing the following imports:

    - 
```
File: contracts/bonding/BondingManager.sol

17    import "@openzeppelin/contracts/utils/math/SafeMath.sol";
```
node_modules/@openzeppelin/contracts/utils/math/SafeMath.sol 
    - 
```
File: contracts/bonding/BondingManager.sol

17    import "@openzeppelin/contracts/utils/math/SafeMath.sol";
```
contracts/bonding/libraries/EarningsPool.sol 
    - 
```
File: contracts/bonding/BondingManager.sol

17    import "@openzeppelin/contracts/utils/math/SafeMath.sol";
```
contracts/libraries/PreciseMathUtils.sol 
    - 
```
File: contracts/bonding/BondingManager.sol

17    import "@openzeppelin/contracts/utils/math/SafeMath.sol";
```
contracts/libraries/MathUtils.sol 
    - 
```
File: contracts/bonding/BondingManager.sol

17    import "@openzeppelin/contracts/utils/math/SafeMath.sol";
```
contracts/snapshots/IMerkleSnapshot.sol 
contracts/snapshots/IMerkleSnapshot.sol

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L17](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L17)


#
Unused imports found in C:/Users/97255/Documents/code4rena/2023-08-livepeer/contracts/bonding/BondingVotes.sol.
Consider removing the following imports:

    - 
```
File: contracts/bonding/BondingVotes.sol

14    import "../rounds/IRoundsManager.sol";
```
node_modules/@openzeppelin/contracts/utils/Arrays.sol 
    - 
```
File: contracts/bonding/BondingVotes.sol

14    import "../rounds/IRoundsManager.sol";
```
contracts/bonding/libraries/EarningsPool.sol 
contracts/bonding/libraries/EarningsPool.sol

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L14](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L14)


#
Unused imports found in C:/Users/97255/Documents/code4rena/2023-08-livepeer/contracts/bonding/libraries/EarningsPoolLIP36.sol.
Consider removing the following imports:

    - 
```
File: contracts/bonding/libraries/EarningsPoolLIP36.sol

7    import "@openzeppelin/contracts/utils/math/SafeMath.sol";
```
node_modules/@openzeppelin/contracts/utils/math/SafeMath.sol 
node_modules/@openzeppelin/contracts/utils/math/SafeMath.sol

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L7](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L7)


#
Unused imports found in C:/Users/97255/Documents/code4rena/2023-08-livepeer/contracts/bonding/libraries/SortedArrays.sol.
Consider removing the following imports:

    - 
```
File: contracts/bonding/libraries/SortedArrays.sol

6    import "@openzeppelin/contracts/utils/Arrays.sol";
```
contracts/libraries/MathUtils.sol 
contracts/libraries/MathUtils.sol

[https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L6](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L6)


</details>

# 
