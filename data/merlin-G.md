# Can pack structs to use fewer storage slots
To optimize gas usage when interacting with the Solidity EVM, it is advisable to encapsulate variables smaller than 32 bytes within a struct. This allows these variables to share the same storage slot, resulting in gas savings when performing storage writes.

The maximum allowable number of rounds is represented by a uint48 value. [clock() function](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingVotes.sol#L137).
The "round" variable can be of type uint48 rather than uint256.
The following structs (`BondingCheckpoint`, `BondingCheckpointsByRound`, `TotalActiveStakeByRound`) can be replaced with this one:

```solidity
 struct BondingCheckpoint {
        uint256 bondedAmount;
        address delegateAddress;
        uint256 delegatedAmount;
        uint48 lastClaimRound;
        uint48 lastRewardRound;
    }

    struct BondingCheckpointsByRound {
        uint48[] startRounds;
        mapping(uint48 => BondingCheckpoint) data;
    }

    struct TotalActiveStakeByRound {
        uint48[] rounds;
        mapping(uint48 => uint256) data;
    }
```


