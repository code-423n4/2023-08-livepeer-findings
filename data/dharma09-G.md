### [G-01] Use storage values instead of the memory one

In `BondingManager.sol` used `delegators[_transcoder]` instead of  `del`  

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L403

```diff
File: contracts/bonding/BondingManager.sol
//@audit delegators[_transcoder] 

-403: uint256 penalty = MathUtils.percOf(delegators[_transcoder].bondedAmount, _slashAmount);
+403: uint256 penalty = MathUtils.percOf(del.bondedAmount, _slashAmount);
```

In `BondingManager.sol` used `lock.withdrawRound` instead of `withdrawRound` in line 255

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L255 

```diff
File: contracts/bonding/BondingManager.sol 
249: function withdrawStake(uint256 _unbondingLockId) external whenSystemNotPaused currentRoundInitialized {
250:        Delegator storage del = delegators[msg.sender];
251:        UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];
252:
253:        require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID");
254:        require(
-255:            lock.withdrawRound <= roundsManager().currentRound(),
+255:            withdrawRound <= roundsManager().currentRound(),
256:            "withdraw round must be before or equal to the current round"
257:        );
258:
259:        uint256 amount = lock.amount;
260:        uint256 withdrawRound = lock.withdrawRound;
261:        // Delete unbonding lock
262:        delete del.unbondingLocks[_unbondingLockId];
263:
264:        // Tell Minter to transfer stake (LPT) to the delegator
265:        minter().trustedTransferTokens(msg.sender, amount);
266:
267:        emit WithdrawStake(msg.sender, _unbondingLockId, amount, withdrawRound);
268:    }
```

### [G-02] Multiple accesses of the same mapping/array key/index should be cached

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves **~42 gas per access** due to not having to recalculate the key's keccak256 hash (Gkeccak256 - **30 gas**) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1193C3-L1196C1

```solidity
File: contracts/bonding/BondingManager.sol
1189: function cumulativeFactorsPool(Transcoder storage _transcoder, uint256 _round)
1190:        internal
1191:        view
1192:        returns (EarningsPool.Data memory pool)
1193:    {
1194:        pool.cumulativeRewardFactor = _transcoder.earningsPoolPerRound[_round].cumulativeRewardFactor; //@audit Cache array
1195:        pool.cumulativeFeeFactor = _transcoder.earningsPoolPerRound[_round].cumulativeFeeFactor;
1196:	
1197:        return pool;
1198:    }
```

```diff
File: contracts/bonding/BondingManager.sol
1189: function cumulativeFactorsPool(Transcoder storage _transcoder, uint256 _round)
1190:        internal
1191:        view
1192:        returns (EarningsPool.Data memory pool)
1193:    {
+             earningsPool storage PoolPerRound =  _transcoder.earningsPoolPerRound[_round];
-1194:        pool.cumulativeRewardFactor = _transcoder.earningsPoolPerRound[_round].cumulativeRewardFactor;
-1195:        pool.cumulativeFeeFactor = _transcoder.earningsPoolPerRound[_round].cumulativeFeeFactor;
+1194:        pool.cumulativeRewardFactor = PoolPerRound.cumulativeRewardFactor; 
+1195:        pool.cumulativeFeeFactor = PoolPerRound.cumulativeFeeFactor;
1196:	
1197:        return pool;
1198:    }
```

### [G-03] Use `calldata` instead of `memory` for immutable arguments

Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L132C3-L150C1

```solidity
File: contracts/treasury/LivepeerGovernor.sol
132: function _execute(
133:        uint256 proposalId,
134:        address[] memory targets, //@audit
135:        uint256[] memory values,
136:        bytes[] memory calldatas,
137:        bytes32 descriptionHash
138:    ) internal override(GovernorUpgradeable, GovernorTimelockControlUpgradeable) {
139:        super._execute(proposalId, targets, values, calldatas, descriptionHash);
140:    }
141:
142:    function _cancel(
143:        address[] memory targets,
144:        uint256[] memory values,
145:        bytes[] memory calldatas,
146:        bytes32 descriptionHash
147:    ) internal override(GovernorUpgradeable, GovernorTimelockControlUpgradeable) returns (uint256) {
148:        return super._cancel(targets, values, calldatas, descriptionHash);
149:    }
```

### [G-04] Amounts should be checked for 0 before calling a transfer

It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```

In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L265

```solidity
File: contracts/bonding/BondingManager.sol
265: minter().trustedTransferTokens(msg.sender, amount);

285: minter().trustedWithdrawETH(_recipient, _amount);

426: minter().trustedTransferTokens(_finder, finderAmount);
```

### [G-05] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

```solidity
File: contracts/bonding/BondingManager.sol
872: uint256 treasuryBalance = livepeerToken().balanceOf(treasury());
```

### [G-06] **Operations which won't underflow don't need to use SafeMath library**

`fees >= _*amount` implies that `````0 < fees-_*amount`, thus this expression won't underflow and can be rewritten as: `delegators[msg.sender].fees = fees - _amount;`

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L282

```solidity
282: delegators[msg.sender].fees = fees.sub(_amount); //See line 281
```

`_amount <= del.bondedAmount *`implies that `0 < del.bondedAmount-*amount`, thus this expression won't underflow and can be rewritten as: `del.bondedAmount = del.bondedAmount - _amount;`

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L767

```solidity
767: del.bondedAmount = del.bondedAmount.sub(_amount); //See line 755
```