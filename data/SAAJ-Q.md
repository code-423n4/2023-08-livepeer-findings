
# Low Risk and Non-Critical Issues

# [L-01] Missing event for important parameter change (13 Instances)
Important parameter or configuration changes should trigger an event to allow being tracked off-chain.

Link to the code:
1.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L18
2.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L47
3.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L71
4.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L54
5.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L132
6.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L142
7.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L218
8.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L167
9.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L207
10.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L215
11.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L223
12.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L232
13.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1459





# [N-01] Empty blocks should be removed (02 Instances)

Avoid using code blocks or use them for some process like emitting events.

Link to the code:
1.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L149
2.	https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L107



# [N-02] Function writing that does not comply with the Solidity Style Guide

All Contracts
Order of functions should help readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered as mentioned in the article i.e.;
constructor
external
public
internal
private
within a grouping, place the view and pure functions last

