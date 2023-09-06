## [L-01] Unsafe erc20 operations
```txt
2023-08-livepeer/contracts/bonding/BondingManager.sol::618 => livepeerToken().transferFrom(msg.sender, address(minter()), _amount);
```
## [L-02] Unspecific compiler version pragma
```txt
2023-08-livepeer/contracts/bonding/BondingManager.sol::2 => pragma solidity ^0.8.9;
2023-08-livepeer/contracts/bonding/BondingVotes.sol::2 => pragma solidity ^0.8.9;
2023-08-livepeer/contracts/bonding/IBondingManager.sol::2 => pragma solidity ^0.8.9;
2023-08-livepeer/contracts/bonding/IBondingVotes.sol::2 => pragma solidity ^0.8.9;
2023-08-livepeer/contracts/bonding/libraries/EarningsPool.sol::2 => pragma solidity ^0.8.9;
2023-08-livepeer/contracts/bonding/libraries/EarningsPoolLIP36.sol::2 => pragma solidity ^0.8.9;
2023-08-livepeer/contracts/bonding/libraries/SortedArrays.sol::2 => pragma solidity ^0.8.9;
2023-08-livepeer/contracts/treasury/GovernorCountingOverridable.sol::2 => pragma solidity ^0.8.9;
2023-08-livepeer/contracts/treasury/IVotes.sol::2 => pragma solidity ^0.8.9;
2023-08-livepeer/contracts/treasury/LivepeerGovernor.sol::2 => pragma solidity ^0.8.9;
2023-08-livepeer/contracts/treasury/Treasury.sol::2 => pragma solidity ^0.8.9;
```