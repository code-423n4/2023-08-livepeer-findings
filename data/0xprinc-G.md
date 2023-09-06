## 1. better to use the `==` instead of `!=` to save some gas
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L24
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L52

```Solidity
        uint256 prevCumulativeRewardFactor = _prevEarningsPool.cumulativeRewardFactor != 0
            ? _prevEarningsPool.cumulativeRewardFactor
            : PreciseMathUtils.percPoints(1, 1);
```

`==` is better than `!=` while compiling the solidity code as the opcode `EQ` is present and `!=` is not in the form of any Opcode, so this introduces the use of other Opcodes like `GT`, `LT` to ensure that the two values compared are not equal. <BR>

The better verion can be
```Solidity
    uint256 prevCumulativeRewardFactor = _prevEarningsPool.cumulativeRewardFactor == 0
            ? PreciseMathUtils.percPoints(1, 1)
            : _prevEarningsPool.cumulativeRewardFactor;

```