### [L-01] treasuryBalance should be > treasuryBalanceCeiling instead of >= as stated in the LIP-92
``Check the current LPT balance on the Livepeer Treasury. If the balance > treasuryBalanceCeiling, set treasuryRewardCutRate = 0 ``
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L873