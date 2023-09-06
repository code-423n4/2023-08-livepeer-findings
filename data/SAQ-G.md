## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Add unchecked{} for subtractions where the operands cannot underflow because of a previous require() or if statement | 1 | - |
| [G-02] | Using fixed bytes is cheaper than using string | 2 | - |
| [G-03] | Not using the named return variable when a function returns, wastes deployment gas | 2 | - |
| [G-04] | Public function not called by the contract should be declared external instead | 4 | - |
| [G-05] | Require() or revert() statements that check input arguments should be at the top of the function | 3 | - |
| [G-06] | Empty blocks should be removed or emit something | 2 | - |
| [G-07] | Using storage instead of memory for structs/arrays saves gas | 8 | - |
| [G-08] | Use constants instead of type(uintx).max | 1 | - |
| [G-09] | Do not calculate  constant | 1 | - |
| [G-10] | require() Should Be Used Instead Of assert() | 2 | - |
| [G-11] | Sort Solidity operations using short-circuit mode | 1 | - |
| [G-12] | 2**<N> should be re-written as type(uint<N>).max | 1 | - |
| [G-13] | Using Bools For Storage Incurs Overhead | 1 | - |
| [G-14] | Use assembly to validate msg.sender | 2 | - |
| [G-15] | No need to evaluate all expressions to know if one of them is true | 1 | - |
| [G-16] | Make 3 event parameters indexed when possible | 4 | - |


## Gas Optimizations  

## [G-01] Add unchecked{} for subtractions where the operands cannot underflow because of a previous require() or if statement  

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
This will stop the check for overflow and underflow so it will save gas

```solidity
file: /contracts/bonding/libraries/SortedArrays.sol

34        if (_array[len - 1] <= _val) {
35            return len - 1;

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L34-L35


## [G-02] Using fixed bytes is cheaper than using string

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

```solidity
file: /contracts/bonding/BondingVotes.sol

115    function name() external pure returns (string memory) {
116        return "Livepeer Voting Power";

/// @audit Since the string "vLPT" consists of 4 characters, each occupying 1 byte, the total size of the string in bytes would be 4 bytes.

122    function symbol() external pure returns (string memory) {
123        return "vLPT";

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L115-L116


## [G-03] Not using the named return variable when a function returns, wastes deployment gas

When you execute a function that returns values in Solidity, the EVM still performs the necessary operations to execute and return those values. This includes the cost of allocating memory and packing the return values. If the returned values are not utilized, it can be seen as wasteful since you are incurring gas costs for operations that have no effect.

```solidity
file: /contracts/bonding/BondingManager.sol

1262     returns (uint256 stake, uint256 fees)

///@audit ' stake, fees '  on line 1262

1286        return (stake, fees);

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1286

### Recommended code 

```solidity
// Remove the function return variables.

1262 returns (uint256 , uint256 )

1286        return (stake, fees);

```

```solidity
file: /contracts/bonding/libraries/EarningsPoolLIP36.sol

///@audit ' cStake, cFees '  on line 76


97        return (cStake, cFees);

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L97


## [G-04] Public function not called by the contract should be declared external instead 

Contracts are allowed to override their parents' functions and change the visibility from external to public and can save gas by doing so.

```solidity
file: /contracts/bonding/BondingManager.sol

923    function pendingFees(address _delegator, uint256 _endRound) public view returns (uint256) {

1027    function getTranscoderEarningsPoolForRound(address _transcoder, uint256 _round)
        public
        view
1030        returns ( 


1089    function getDelegatorUnbondingLock(address _delegator, uint256 _unbondingLockId)
        public
        view
1092        returns (uint256 amount, uint256 withdrawRound)    


```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L923


```solidity
file: /contracts/treasury/GovernorCountingOverridable.sol

76    function COUNTING_MODE() public pure virtual override returns (string memory) {

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L76


```solidity
file: /contracts/treasury/LivepeerGovernor.sol

78    function quorumDenominator() public view virtual override returns (uint256) {

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L78


## [G-05] Require() or revert() statements that check input arguments should be at the top of the function  

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

```solidity
file: /contracts/bonding/BondingManager.sol

/// @audit the bellow require statements should be the top of thire functions.

253        require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID");

582        require(!isRegisteredTranscoder(_owner), "registered transcoders can't delegate towards other addresses");

1573        require(isValidUnbondingLock(_delegator, _unbondingLockId), "invalid unbonding lock ID");

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L253


## [G-06] Empty blocks should be removed or emit something

The gas cost of an empty constructor block in Solidity is typically in the range of 200-500 gas units.
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

```solidity
file: /contracts/bonding/BondingManager.sol

149    constructor(address _controller) Manager(_controller) {}

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L149


```solidity
file: /contracts/bonding/BondingVotes.sol

107    constructor(address _controller) Manager(_controller) {}

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L107


## [G-07] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
file: /contracts/treasury/Treasury.sol

18        address[] memory proposers,
19        address[] memory executors,

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/Treasury.sol#L18-L19


```solidity
file: /contracts/treasury/LivepeerGovernor.sol

134        address[] memory targets,
135       uint256[] memory values,
136        bytes[] memory calldatas,


143        address[] memory targets,
144        uint256[] memory values,
145        bytes[] memory calldatas,

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L134-L136


## [G-08] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: /contracts/treasury/GovernorCountingOverridable.sol

22    error InvalidVoteType(uint8 voteType);

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L22


## [G-09] Do not calculate  constant

When you define a constant in Solidity, the compiler can calculate its value at compile-time and replace all references to the constant with its computed value. This can be helpful for readability and reducing the size of the compiled code, but it can also increase gas usage at runtime.

```solidity
file: /contracts/bonding/BondingManager.sol

32    uint256 constant MAX_FUTURE_ROUND = 2**256 - 1;

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L32


## [G-10] require() Should Be Used Instead Of assert()

assert() will consume all remaining gas and should only be used for conditions that are truly impossible to fail.

```solidity
file: /contracts/bonding/libraries/SortedArrays.sol

41        assert(upperIdx < len);

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L41


```solidity
file: /contracts/treasury/GovernorCountingOverridable.sol

206         assert(delegateSupport == VoteType.Abstain);

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L206


## [G-11] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows
 f(x) || g(y) 
 f(x) && g(y)
```

```solidity
file: /contracts/bonding/BondingManager.sol

343        if (prevEarningsPool.cumulativeRewardFactor == 0 && lastRewardRound == currentRound) {

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L343


## [G-12] 2**<N> should be re-written as type(uint<N>).max

Earlier versions of solidity can use uint<n>(-1) instead. Expressions not including the - 1 can often be re-written to accomodate the change (e.g. by using a > rather than a >=, which will also save some gas)

```solidity
file: /contracts/bonding/BondingManager.sol

32    uint256 constant MAX_FUTURE_ROUND = 2**256 - 1;

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L32


## [G-13] Using Bools For Storage Incurs Overhead

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.

```solidity
file: /contracts/treasury/GovernorCountingOverridable.sol

38        bool hasVoted;

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L38


## [G-14] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```solidity
file: contracts/bonding/BondingManager.sol

559    if (msg.sender != _owner && msg.sender != l2Migrator()) {
    
```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L559

```solidity
file:  contracts/bonding/BondingVotes.sol

554    if (msg.sender != address(bondingManager())) {

```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L554


## [G-15] No need to evaluate all expressions to know if one of them is true

When we have a code expressionA || expressionB if expressionA is true then expressionB will not be evaluated and gas saved;

```solidity
file: contracts/bonding/BondingManager.sol

499  require(
            !isActiveTranscoder(msg.sender) || t.lastRewardRound == currentRound,
            "caller can't be active or must have already called reward for the current round"
503        );

```

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L499-L503


## [G-16] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
file: /contracts/bonding/IBondingManager.sol

10  event TranscoderActivated(address indexed transcoder, uint256 activationRound);

11  event TranscoderDeactivated(address indexed transcoder, uint256 deactivationRound);

13  event Reward(address indexed transcoder, uint256 amount);

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L10

```solidity
file: /contracts/bonding/IBondingVotes.sol

27    event DelegatorBondedAmountChanged(address indexed delegate, uint256 previousBondedAmount, uint256 newBondedAmount);

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingVotes.sol#L27
