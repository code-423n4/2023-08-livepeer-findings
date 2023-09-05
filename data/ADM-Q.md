###### [L01] Treasury Ceiling can be exceeded.

The amount of rewards sent to the treasury is based on treasuryRewardCutRate. When the ceiling is hit this value is intended to be set to 0, resulting in no more tokens being sent to the treasury until either the ceiling is raised or the number of tokens in the treasury drops and an admin alters the treasuryRewardCutRate back to a value > 0.
The issue arises in that the check to change this value to 0 is done before transfering tokens to the treasury. Say Treasury ceiling is 100 and there are currently 90 tokens in the treasury when rewardWithHint is called it will check if the treasury balance is greater or equal than the ceiling which it is currently not so the treasuryRewardCutRate will remain the same.
```Solidity 
    uint256 treasuryBalance = livepeerToken().balanceOf(treasury());
    if (treasuryBalance >= treasuryBalanceCeiling && nextRoundTreasuryRewardCutRate > 0) {
```
Now say treasuryRewards which is calculated later in the function equals 10, it will result in 10 tokens being sent to the treasury which will now total 100. The ceiling has now been hit and treasuryRewardCutRate should be set to 0 however this wont happen until rewardWithHint is called again which can result in more treasuryRewards being sent to the treasury address than the treasuryBalanceCeiling.

I recommend moving the block of code that checks the [treasury balance](https://github.com/code-423n4/2023-08-livepeer/blob/bcf493b98d0ef835e969e637f25ea51ab77fabb6/contracts/bonding/BondingManager.sol#L871-L877) to after the block of code that transfers tokens to the [treasury](https://github.com/code-423n4/2023-08-livepeer/blob/bcf493b98d0ef835e969e637f25ea51ab77fabb6/contracts/bonding/BondingManager.sol#L881-L890).

