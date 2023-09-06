
## Low Risk Issues

|                        Number                         | Issues                                  | Instances |
| :---------------------------------------------------: | :-------------------------------------- | :-------: |
|  [L-01](#l-01-no-limits-when-setting-minmax-amounts)  | No limits when setting min/max amounts  |     7     |
| [L-02](#l-02-pausing-withdraw-may-be-unfair-to-users) | Pausing withdraw may be unfair to users |     2     |

Total: 10 over 3 issues

## Non-Critical Issues

|                           Number                            | Issues                                        | Instances |
| :---------------------------------------------------------: | :-------------------------------------------- | :-------: |
|   [N-01](#n-01-function-must-not-be-longer-than-50-lines)   | Function must not be longer than 50 lines     |     5     |
|              [N-02](#n-02-remove-unused-code)               | Remove unused code                            |     4     |
| [N-03](#n-13-functions-that-alter-state-should-emit-events) | Functions that alter state should emit events |     2     |
|    [N-04](#n-04-setters-should-have-initial-value-check)    | Setters should have initial value check       |     3     |
| [N-05](#n-05-missing-event-for-critical-parameters-change)  | Missing event for critical parameters change  |     2     |
|        [N-06](#n-06-lack-of-space-near-the-operator)        | Lack of space near the operator               |     1     |

Total: 22 over 51 issues

## [L-01] No limits when setting min/max amounts

It is important to ensure that the min/max amounts are set to a reasonable value.

<details>
<summary><i>There are 2 instance of this issue:</i></summary>

```solidity
File: contracts/bonding/BondingManager.sol

155:     function setUnbondingPeriod(uint64 _unbondingPeriod) external onlyControllerOwner {
156:         unbondingPeriod = _unbondingPeriod;
157:
158:         emit ParameterUpdate("unbondingPeriod");
159:     }

176:     function setTreasuryBalanceCeiling(uint256 _ceiling) external onlyControllerOwner {
177:         treasuryBalanceCeiling = _ceiling;
178:
179:         emit ParameterUpdate("treasuryBalanceCeiling");
180:     }

```

[155](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L155-L159), [176](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L176-L180).

</details>

## [L-02] Pausing withdraw may be unfair to users

Users should be able to withdraw their funds at any time.

<details>
<summary><i>There are 2 instance of this issue:</i></summary>

```solidity
File: contracts/bonding/BondingManager.sol

249:     function withdrawStake(uint256 _unbondingLockId) external whenSystemNotPaused currentRoundInitialized {
250:         Delegator storage del = delegators[msg.sender];
251:         UnbondingLock storage lock = del.unbondingLocks[_unbondingLockId];
252:
253:         require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID");
254:         require(
255:             lock.withdrawRound <= roundsManager().currentRound(),
256:             "withdraw round must be before or equal to the current round"
257:         );
258:
259:         uint256 amount = lock.amount;
260:         uint256 withdrawRound = lock.withdrawRound;
261:         // Delete unbonding lock
262:         delete del.unbondingLocks[_unbondingLockId];
263:
264:         // Tell Minter to transfer stake (LPT) to the delegator
265:         minter().trustedTransferTokens(msg.sender, amount);
266:
267:         emit WithdrawStake(msg.sender, _unbondingLockId, amount, withdrawRound);
268:     }

273:     function withdrawFees(address payable _recipient, uint256 _amount)
274:         external
275:         whenSystemNotPaused
276:         currentRoundInitialized
277:         autoClaimEarnings(msg.sender)
278:     {
279:         require(_recipient != address(0), "invalid recipient");
280:         uint256 fees = delegators[msg.sender].fees;
281:         require(fees >= _amount, "insufficient fees to withdraw");
282:         delegators[msg.sender].fees = fees.sub(_amount);
283:
284:         // Tell Minter to transfer fees (ETH) to the address
285:         minter().trustedWithdrawETH(_recipient, _amount);
286:
287:         emit WithdrawFees(msg.sender, _recipient, _amount);
288:     }

```

[249](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L249-L268), [273](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L273-L288).

</details>

## [N-01] Function must not be longer than 50 lines

<details>
<summary><i>There are 5 instance of this issue:</i></summary>

```solidity
File: contracts/bonding/BondingManager.sol

302:     function updateTranscoderWithFees(

537:     function bondForWithHint(

679:     function transferBond(

842:     function rewardWithHint(address _newPosPrev, address _newPosNext)

1500:     function updateDelegatorWithEarnings(

```

[302](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L302), [537](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L537), [679](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L679), [842](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L842), [1500](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1500).

</details>

## [N-02] Remove unused code

If code is not used, it should be removed to reduce the size of the contract and increase readability.

<details>
<summary><i>There are 4 instance of this issue:</i></summary>

```solidity
File: contracts/bonding/BondingManager.sol

307:         // Silence unused param compiler warning

453:         // Silence unused param compiler warning

909:         // Silence unused param compiler warning

924:         // Silence unused param compiler warning

```

[307](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L307), [453](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L453), [909](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L909), [924](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L924).

</details>

## [N-03] Functions that alter state should emit events

Functions that alter state should emit events to notify users of the state change. This is especially important for functions that alter state and do not return a value.

<details>
<summary><i>There are 2 instance of this issue:</i></summary>

```solidity
File: contracts/bonding/BondingManager.sol

1307:     function increaseTotalStakeUncheckpointed(

1352:     function decreaseTotalStake(

```

[1307](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1307), [1352](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1352).

</details>

## [N-04] Setters should have initial value check

Setters should have initial value check to prevent assigning wrong value to the variable. Assignment of wrong value can lead to unexpected behavior of the contract.

<details>
<summary><i>There are 3 instance of this issue:</i></summary>

```solidity
File: contracts/bonding/BondingManager.sol

154:     /// @audit Not validated: _unbondingPeriod

155:     function setUnbondingPeriod(uint64 _unbondingPeriod) external onlyControllerOwner {
156:         unbondingPeriod = _unbondingPeriod;
157:
158:         emit ParameterUpdate("unbondingPeriod");
159:     }

175:     /// @audit Not validated: _ceiling

176:     function setTreasuryBalanceCeiling(uint256 _ceiling) external onlyControllerOwner {
177:         treasuryBalanceCeiling = _ceiling;
178:
179:         emit ParameterUpdate("treasuryBalanceCeiling");
180:     }

185:     /// @audit Not validated: _numActiveTranscoders

186:     function setNumActiveTranscoders(uint256 _numActiveTranscoders) external onlyControllerOwner {
187:         transcoderPool.setMaxSize(_numActiveTranscoders);
188:
189:         emit ParameterUpdate("numActiveTranscoders");
190:     }

```

[155](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L155-L159), [176](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L176-L180), [186](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L186-L190).

</details>
## [N-21] Function name should be in mixedCase

<details>
<summary><i>There are 2 instance of this issue:</i></summary>

```solidity
File: contracts/bonding/BondingVotes.sol

145:     function CLOCK_MODE() external pure returns (string memory) {

```

[145](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L145).

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

76:     function COUNTING_MODE() public pure virtual override returns (string memory) {

```

[76](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L76).

</details>

## [N-05] Missing event for critical parameters change

It is important to log critical parameters change to be able to track the state of the contract.

<details>
<summary><i>There are 2 instance of this issue:</i></summary>

```solidity
File: contracts/bonding/BondingManager.sol

1307:     function increaseTotalStakeUncheckpointed(
1308:         address _delegate,
1309:         uint256 _amount,
1310:         address _newPosPrev,
1311:         address _newPosNext
1312:     ) internal {
1313:         Transcoder storage t = transcoders[_delegate];
1314:
1315:         uint256 currStake = transcoderTotalStake(_delegate);
1316:         uint256 newStake = currStake.add(_amount);
1317:
1318:         if (isRegisteredTranscoder(_delegate)) {
1319:             uint256 currRound = roundsManager().currentRound();
1320:             uint256 nextRound = currRound.add(1);
1321:
1322:             // If the transcoder is already in the active set update its stake and return
1323:             if (transcoderPool.contains(_delegate)) {
1324:                 transcoderPool.updateKey(_delegate, newStake, _newPosPrev, _newPosNext);
1325:                 nextRoundTotalActiveStake = nextRoundTotalActiveStake.add(_amount);
1326:
1327:                 // currStake (the transcoder's delegatedAmount field) will reflect the transcoder's stake from lastActiveStakeUpdateRound
1328:                 // because it is updated every time lastActiveStakeUpdateRound is updated
1329:                 // The current active total stake is set to currStake to ensure that the value can be used in updateTranscoderWithRewards()
1330:                 // and updateTranscoderWithFees() when lastActiveStakeUpdateRound > currentRound
1331:                 if (t.lastActiveStakeUpdateRound < currRound) {
1332:                     t.earningsPoolPerRound[currRound].setStake(currStake);
1333:                 }
1334:
1335:                 t.earningsPoolPerRound[nextRound].setStake(newStake);
1336:                 t.lastActiveStakeUpdateRound = nextRound;
1337:             } else {
1338:                 // Check if the transcoder is eligible to join the active set in the update round
1339:                 tryToJoinActiveSet(_delegate, newStake, nextRound, _newPosPrev, _newPosNext);
1340:             }
1341:         }
1342:
1343:         // Increase delegate's delegated amount
1344:         delegators[_delegate].delegatedAmount = newStake;
1345:     }

1352:     function decreaseTotalStake(
1353:         address _delegate,
1354:         uint256 _amount,
1355:         address _newPosPrev,
1356:         address _newPosNext
1357:     ) internal autoCheckpoint(_delegate) {
1358:         Transcoder storage t = transcoders[_delegate];
1359:
1360:         uint256 currStake = transcoderTotalStake(_delegate);
1361:         uint256 newStake = currStake.sub(_amount);
1362:
1363:         if (transcoderPool.contains(_delegate)) {
1364:             uint256 currRound = roundsManager().currentRound();
1365:             uint256 nextRound = currRound.add(1);
1366:
1367:             transcoderPool.updateKey(_delegate, newStake, _newPosPrev, _newPosNext);
1368:             nextRoundTotalActiveStake = nextRoundTotalActiveStake.sub(_amount);
1369:
1370:             // currStake (the transcoder's delegatedAmount field) will reflect the transcoder's stake from lastActiveStakeUpdateRound
1371:             // because it is updated every time lastActiveStakeUpdateRound is updated
1372:             // The current active total stake is set to currStake to ensure that the value can be used in updateTranscoderWithRewards()
1373:             // and updateTranscoderWithFees() when lastActiveStakeUpdateRound > currentRound
1374:             if (t.lastActiveStakeUpdateRound < currRound) {
1375:                 t.earningsPoolPerRound[currRound].setStake(currStake);
1376:             }
1377:
1378:             t.lastActiveStakeUpdateRound = nextRound;
1379:             t.earningsPoolPerRound[nextRound].setStake(newStake);
1380:         }
1381:
1382:         // Decrease old delegate's delegated amount
1383:         delegators[_delegate].delegatedAmount = newStake;
1384:     }

```

[1307](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1307-L1345), [1352](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1352-L1384).

</details>

## [N-06] Lack of space near the operator

Operators [should](https://docs.soliditylang.org/en/latest/style-guide.html#other-recommendations) be surrounded by a single space on either side.

<details>
<summary><i>There are 1 instance of this issue:</i></summary>

```solidity
File: contracts/bonding/BondingManager.sol

32:     uint256 constant MAX_FUTURE_ROUND = 2**256 - 1;

```

[32](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L32).

</details>

