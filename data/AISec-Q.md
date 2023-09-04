## Impact
Missing constructor sanity checks

The implementation of constructor() functions are missing some sanity checks.


## Proof of Concept

Missing sanity check for != address(0)

There're no sanity checks for the constructor argument _controller.

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol?plain=1#L43-L45
```
    constructor(address _controller) Manager(_controller) {
        _disableInitializers();
    }
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol?plain=1#L149
```
    constructor(address _controller) Manager(_controller) {}
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol?plain=1#L107
```
    constructor(address _controller) Manager(_controller) {}
```

## Tools Used
Manual code review

## Recommended Mitigation Steps
To reduce possible errors and make the code more robust, consider adding sanity checks where needed.