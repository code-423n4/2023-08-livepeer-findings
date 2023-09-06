## [L-01] Unsafe erc20 operations
```txt
2023-08-livepeer/contracts/bonding/BondingManager.sol::618 => livepeerToken().transferFrom(msg.sender, address(minter()), _amount);
```