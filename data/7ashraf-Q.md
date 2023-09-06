# EarningsPoolLIP36.sol
-@line 18 updateCumulativeFeeFactor, the method takes parameter _fee, and accepts negative values as an input which may result in weird behavior.

# BondingVotes.sol
-@line 155 getVotes: accepts address 0
-@line 274: if the check fails on 'previous' value, the previous value is never initialized
-The value clock() is very critical, any change to the value may result in stopping the while functionality in the contract 

# BondingManager.sol
-@line 689: need to check for oldDel and newDel values to see if exist and are valid or not before using them in the rest of the method.