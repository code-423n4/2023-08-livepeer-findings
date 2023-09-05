1. The value of PERC_DIVISOR in the MathUtils library used in BondingManager.sol and the protocol in general should be rewritten to make it more readable - https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/libraries/MathUtils.sol#L10
uint256 public constant PERC_DIVISOR = 1000_000; 
instead of uint256 public constant PERC_DIVISOR = 1000000; 