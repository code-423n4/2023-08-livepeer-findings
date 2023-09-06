## [Q-01] - Potential Code Redundancy

**Description:** The code sets default values for _startPool.cumulativeRewardFactor and _endPool.cumulativeRewardFactor when they are zero. However, these default values are always the same, resulting in potential code redundancy.

**Location**

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L77C4-L85C10

**Code Snippet:**

// If the start cumulativeRewardFactor is 0 set the default value to PreciseMathUtils.percPoints(1, 1)
if (_startPool.cumulativeRewardFactor == 0) {
    _startPool.cumulativeRewardFactor = PreciseMathUtils.percPoints(1, 1);
}

// If the end cumulativeRewardFactor is 0 set the default value to PreciseMathUtils.percPoints(1, 1)
if (_endPool.cumulativeRewardFactor == 0) {
    _endPool.cumulativeRewardFactor = PreciseMathUtils.percPoints(1, 1);
}
**Suggested Fix:**
 Since both conditions set the same default value, you can eliminate redundancy by setting it once before the conditions:

_startPool.cumulativeRewardFactor = _startPool.cumulativeRewardFactor == 0 ? PreciseMathUtils.percPoints(1, 1) : _startPool.cumulativeRewardFactor;
_endPool.cumulativeRewardFactor = _endPool.cumulativeRewardFactor == 0 ? PreciseMathUtils.percPoints(1, 1) : _endPool.cumulativeRewardFactor;


## [Q-02] - Unnecessary Check
**Description:** This function pushes a value into an already sorted array. It checks if the value to be pushed is less than the last element of the array and reverts with the DecreasingValues error if so. However, this check seems unnecessary because the function is intended for pushing values in increasing order, and the code logic already ensures this. The check can be removed.

**Location**

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L71C13-L78C14

**Code Snippet:**

// Original code
if (val < last) {
    revert DecreasingValues(val, last);
}

// Suggested fix (remove the check)
// No need to check for val < last because the function assumes values are pushed in increasing order.

**Suggested Fix:**
 Remove the check for val < last as it is redundant.