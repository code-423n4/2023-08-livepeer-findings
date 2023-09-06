
|                                   Number                                   | Issues                                                       | Instances | Total Gas Saved |
| :------------------------------------------------------------------------: | :----------------------------------------------------------- | :-------: | :-------------: |
| [G-01](#g-01-shortcircuit-rules-can-be-be-used-to-optimize-some-gas-usage) | Shortcircuit rules can be be used to optimize some gas usage |     1     |      2100       |
| [G-02](#g-02-avoid-wasting-gas-on-unnecessary-copying-of-local-variables)  | Avoid wasting gas on unnecessary copying of local variables  |     3     |        -        |
|           [G-03](#g-04-using-bools-for-storage-incurs-overhead)            | Using `bool`s for storage incurs overhead                    |     1     |      17100      |
|     [G-04](#g-04-setters-should-prevent-re-setting-of-the-same-value)      | Setters should prevent re-setting of the same value          |     3     |      6300       |

Total: 8 over 4 issues


## [G-01] Shortcircuit rules can be be used to optimize some gas usage

Some conditions may be reordered to save an `SLOAD` (**2100 gas**), as we avoid reading state variables when the first part of the condition fails (with `&&`), or succeeds (with `||`).

<details>
<summary><i>There are 1 instance of this issue:</i></summary>

```solidity
File: contracts/bonding/BondingManager.sol

873:             if (treasuryBalance >= treasuryBalanceCeiling && nextRoundTreasuryRewardCutRate > 0) {

```

[873](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L873)

</details>

## [G-02] Avoid wasting gas on unnecessary copying of local variables

The input variable was copied to a local variable and never modified. Consider using the input variable directly to reduce overhead and save gas.

<details>
<summary><i>There are 3 instance of this issue:</i></summary>

```solidity
File: contracts/bonding/BondingManager.sol

369:         uint256 transcoderCommissionFees = _fees.sub(delegatorsFees);

1506:         uint256 startRound = _lastClaimRound.add(1);

```

[369](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L369), [1316](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1316), [1361](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1361), [1506](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1506).

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

181:         uint256 timepoint = proposalSnapshot(_proposalId);

```

[181](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L181).

</details>

## [G-03] Using `bool`s for storage incurs overhead

Use `uint256(1)` and `uint256(2)` for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [example](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

<details>
<summary><i>There is 1 instance of this issue:</i></summary>

```solidity
File: contracts/treasury/GovernorCountingOverridable.sol

148:         voter.hasVoted = true;

```

[148](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L148).

</details>

## [G-04] Setters should prevent re-setting of the same value

<details>
<summary><i>There are 3 instance of this issue:</i></summary>

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

1176:     function _setTreasuryRewardCutRate(uint256 _cutRate) internal {
1177:         require(PreciseMathUtils.validPerc(_cutRate), "_cutRate is invalid precise percentage");
1178:
1179:         nextRoundTreasuryRewardCutRate = _cutRate;
1180:
1181:         emit ParameterUpdate("nextRoundTreasuryRewardCutRate");
1182:     }

```

[155](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L155-L159), [176](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L176-L180), [1176](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1176-L1182).

</details>
