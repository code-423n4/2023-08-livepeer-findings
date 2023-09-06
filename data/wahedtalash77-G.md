|     |     |
| --- | --- |
| \[G-01\] | Do not calculate constants |
| \[G-02\] | Use `assembly` to check for `address(0)` |
| \[G-03\] | Remove SafeMath library |
| \[G-04\] |  Gas optimizations - storage over memory |
| \[G-05\] |  Using calldata instead of memory for read-only arguments in external functions saves gas |
| \[G-06\] | `+=` Costs More Gas Than `=+` For State Variables |
| \[G-07\] | `Booleans` use 8 bits while you only need 1 bit |
| \[G-08\] | Optimize names to save gas |
| \[G-09\] | Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement |
| \[G-10\] | Make 3 event parameters indexed when possible |
| \[G-11\] | Events are not indexed |
| \[G-12\] | State variables should be cached in stack variables rather than re-reading them from storage |
| \[G‑13\] | choose a proper version of Solidity |
| \|\[G-14 | Use assembly for math (add, sub, mul, div) |

## \[G-01\] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```
uint256 constant MAX_FUTURE_ROUND = 2**256 - 1;
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L32

## \[G-02\] Use `assembly` to check for `address(0)`

```
if (newDel.delegateAddress == address(0) && newDel.bondedAmount == 0) {
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L719

```
del.delegateAddress = address(0);
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L771

```
require(_recipient != address(0), "invalid recipient");
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L279

```
if (_finder != address(0)) {
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L424

### Mitigation

Using `assembly` to check for the zero address can result in significant gas savings compared to using a Solidity expression; especially if the check is performed frequently or in a loop. However, it’s important to note that using `assembly` can make the code less readable and harder to maintain, so it should be used judiciously and with caution.

\[G-3\] `variable == false` instead of `!variable`.
==a bit cheapier when you replace:==

```
if (!transcoderPool.contains(msg.sender)) {
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L507

## \[G-03\] Remove SafeMath library

Gas saved: 2.1K for each tx that uses SafeMath + 100 units per call.

It seems like safe math doesn’t serve any purpose here and it can simply be replaced with normal math operands. The usage of library is pretty expensive since it makes a delegate call to an external address.

## [G-04\] Gas optimizations - storage over memory

In `BondingManager.sol `the functions are using memory keyword, but using storage would reduce the gas cost

```
function delegatorCumulativeStakeAndFees(
        Transcoder storage _transcoder,
        uint256 _startRound,
        uint256 _endRound,
        uint256 _stake,
        uint256 _fees
    ) internal view returns (uint256 cStake, uint256 cFees) {
        // Fetch start cumulative factors
        EarningsPool.Data memory startPool = cumulativeFactorsPool(_transcoder, _startRound);
        // Fetch end cumulative factors
        EarningsPool.Data memory endPool = latestCumulativeFactorsPool(_transcoder, _endRound);

        return EarningsPoolLIP36.delegatorCumulativeStakeAndFees(startPool, endPool, _stake, _fees);
    }
```

## \[G‑05\] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the `calldata` to the `memory` index. Each iteration of this for-loop costs at least 60 gas (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having `memory` arguments, it’s still valid for implementation contracs to use `calldata` arguments instead.

If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it’s still more gass-efficient to use `calldata` when the `external` function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one.

```
function name() external pure returns (string memory) {
        return "Livepeer Voting Power";
    }

    /**
     * @notice Returns the symbol of the token underlying the voting power.
     */
    function symbol() external pure returns (string memory) {
        return "vLPT";
    }
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingVotes.sol#L115C3-L124C6

```
function CLOCK_MODE() external pure returns (string memory) {
        return "mode=livepeer_round";
    }
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingVotes.sol#L145C3-L147C6

```
contract Treasury is Initializable, TimelockControllerUpgradeable {
    function initialize(
        uint256 minDelay,
        address[] memory proposers,
        address[] memory executors,
        address admin
    ) external initializer {
        __TimelockController_init(minDelay, proposers, executors, admin);
    }
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/Treasury.sol#L15C1-L23C6

## \[G-06\] `+=` Costs More Gas Than `=+` For State Variables

### Mitigation

When you use the `+=` operator on a state variable, the EVM has to perform three operations:

- Load the current value of the state variable.
- Add the new value to it.
- Then store the result back in the state variable.

On the other hand, when you use the `=` operator and then add the values separately, the EVM only needs to perform two operations:

- Load the current value of the state variable.
- Add the new value to it.

Better use `=+` For State Variables.

```
if (support == VoteType.Against) {
            tally.againstVotes += _weight;
        } else if (support == VoteType.For) {
            tally.forVotes += _weight;
        } else {
            tally.abstainVotes += _weight;
        }
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L153C6-L159C10

## \[G-07\] `Booleans` use 8 bits while you only need 1 bit

Under the hood of solidity, `Booleans (bool)` are uint8 which means they use 8 bits of storage. A `Boolean` can only have two values: True or False. This means that you can store a `Boolean` in only a single bit. You can pack 256 `Booleansin` a single word. The easiest way is to take a uint256 variable and use all 256 bits of it to represent individual `Booleans`.

### [Reference](https://blog.polymath.network/solidity-tips-and-tricks-to-save-gas-and-reduce-bytecode-size-c44580b218e6)

```
bool isTranscoder = _account == delegate;
        if (isTranscoder) {
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L184

## \[G-08\] Optimize names to save gas

> Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

> See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).

### Recommended Mitigation Steps

> Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas.

## \[G‑09\] Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement

```
if (delegateSupport == VoteType.Against) {
                _tally.againstVotes -= _weight;
            } else if (delegateSupport == VoteType.For) {
                _tally.forVotes -= _weight;
            } else {
                assert(delegateSupport == VoteType.Abstain);
                _tally.abstainVotes -= _weight;
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/treasury/GovernorCountingOverridable.sol#L201C13-L207C48

## \[G-10\] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

### [Reference](https://code4rena.com/reports/2023-01-drips#g-05-make-3-event-parameters-indexed-when-possible)

```
event TranscoderActivated(address indexed transcoder, uint256 activationRound);
    event TranscoderDeactivated(address indexed transcoder, uint256 deactivationRound);
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L10C2-L11C88

```
event Reward(address indexed transcoder, uint256 amount);
```

https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/IBondingManager.sol#L13

## <ins>\[G-11\] Events are not indexed</ins>

The emitted events are not indexed, making off-chain scripts such as front-ends of dApps difficult to filter the events efficiently.

Recommend adding the `indexed` keyword in each event,

https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol

## \[G-12\] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

> all Contest

## \[G‑13\] choose a proper version of Solidity

Use a Solidity version of at least 0.8.2 to get simple compiler automatic inlining.

Use a Solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.

Use a Solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.

Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.