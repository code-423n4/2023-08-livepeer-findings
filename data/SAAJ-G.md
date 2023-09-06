 
# Gas Optimizations Report

This report focuses on Livepeer Protocol contest, in context of various improvements that can be made in terms of gas cost.

Some of the opportunities identified for improving gas efficiency throughout the codebase of Livepeer protocol are categorised into 06 main areas; with further multiple instances in each of the category.

# Summary

[G-01] Increase number of optimisation runs (01 Instances)
[G-02] Avoid storage value emit (03 Instances)
[G-03] Storage over memory (07 Instances)
[G 04] Consolidating library functions can save gas by preventing external calls (14 Instances)
[G-05] Use uint256(1)/uint256(2) instead for true and false boolean states (04 Instances)
[G-06] Struct can be packed into fewer storage slots (02 Instances)


# [G-01] Increase number of optimisation runs (01 Instances)
Optimisation runs should be changed to 1000 instead of 200. This helps in removing unused opcodes which are the main element of gas consumption.
The gas of contract deployment will be comparably high but will cost less gas when functions are executed.

Link to the Code:
1.	https://github.com/code-423n4/2023-08-livepeer/blob/main/hardhat.config.ts#L43


# [G-02] Avoid storage value emit (03 Instances)
Emitting storage value requires reading from storage is expensive, instead using the argument passed as new value is more gas efficient.

Link to the Code:
1.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L158
2.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L179
3.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1181



# [G-03] Storage over memory (07 Instances)
Some functions are using memory to read state variables when using storage is more gas efficient.

Link to the Code:
1.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L322
2.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1246
3.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1248
4.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1468
5.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L273
6.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L278
7.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L464


# [G 04] Consolidating library functions can save gas by preventing external calls (14 Instances)
Libraries of functions commonly used across different contracts come at the increased cost of gas usage at runtime because of the external calls.
Consider moving all the library functions internal to contract or to a single library to save gas from external calls.

Link to the code:
1.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L5
2.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L4
3.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L8
4.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L9
5.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L11
6.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L12
7.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L6
8.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L7
9.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L8
10.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L9
11.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L10
12.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L7
13.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L8
14.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L9



# [G-05] Use uint256(1)/uint256(2) instead for true and false boolean states (04 Instances)

Boolean for storage if not used, saves Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

Link to the Code:
1.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L148
2.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L370
3.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L398
4.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L399


# [G-06] Struct can be packed into fewer storage slots (02 Instances)
Structs with proper slot allocation and similar data types when clustered together are more efficient.	
	
Link to the Code:
1.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L38
2.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L24

