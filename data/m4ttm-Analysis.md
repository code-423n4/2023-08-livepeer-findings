## 1. Audit Approach

|     |     |     |
| --- | --- | --- |
| Step | Task | Details |
| 1   | Run Tests | Tests run successfully |
| 2   | Coverage | 100% test coverage for contracts in audit scope |
| 3   | Slither | Reviewed Slither results, no vulnerabilities discovered |
| 4   | Surya | Generate graphs to understand the overall project structure. Provided an initial insight to the contract inheritance and function call flow |
| 5   | Solidity Metrics | Generate metrics reports to obtain initial insight on the codebase, noting areas of potential concern |
| 6   | Code Review | Line by line code review |
| 7   | Test Review | Review of each test and it's purpose |

## 2. Mechanism Summary

Livepeer is a decentralised video streaming solution, where "transcoders" are responsible for changing the bitrate of a video stream according to the users needs, this is a computationally expensive operation and they receive rewards for doing so. Token holders can "bond" their tokens and delegate them to a chosen transcoder and receive rewards in return. Protocol Bonding rewards are managed in `BondingManager` with `BondingVotes` being used to store checkpoints to provide an [ERC-5805](https://eips.ethereum.org/EIPS/eip-5805) compliant voting with delegation mechanism. A library to assist in finding the lower bound in an array is used for querying checkpoints, and implemented in `SortedArrays`. A custom staking rewards mechanism is specified in [LIP-36](https://github.com/livepeer/LIPs/blob/master/LIPs/LIP-36.md) and implemented in `EarningsPoolLIP36`. The governor implementation is `LivepeerGovernor` and is built using OpenZeppelin governance libraries to build a custom governance solution, with upgradability and timelock functionality. This inherits from `GovernorCountingOverridable` which allows a delegator to override the transcoders votes, should they choose to do so. Controlled by the governance is the `Treasury` which holds funds and executes proposals.

## 3. Centralisation Risks

The Livepeer protocol is DAO based, with a good [distribution of token holders](https://etherscan.io/token/tokenholderchart/0x58b6a8a3302369daec383334672404ee733ab239). Token holders can vote on changes to the protocol in a decentralised manner without a single source of ownership. This minimises the risk of centralisation. A timelock is used when executing proposals, reducing the risk of malicious proposals causing damage.

## 4. Systemic Risks

There is a risk of malicious proposals being voted on and breaking the protocol or losing funds. Changes to the protocol may have unintended side effects, not spotted by voters. The consequences of any changes should be well considered before implementing them.

## 5. Quality Analysis

### 5.1 Codebase

The code is well structured and organised into separate contracts, each with a clear purpose. NatSpec comments are well used throughout the codebase. The [automated findings](https://github.com/code-423n4/2023-08-livepeer/blob/main/bot-report.md) show that standard conventions, such as prefixing internal/private functions with an underscore, or function ordering according to the Solidity Style Guide are not well followed. There are also many small changes identified which would result in saving gas.

### 5.2 Documentation

NatSpec comments are present throughout the code, although are incomplete in places. In depth details of how the contracts work can be found in the [technical spec](https://github.com/livepeer/LIPs/blob/master/assets/lip-89/treasury_technical_spec.md). This could be improved further with diagrams outlining the workings of the core functionality.

### 5.3 Tests

Full test coverage is achieved for the contracts in scope, making use of both Hardhat and Foundry. Tests are well written and clearly structured. Testing could be further improved with the use of fuzz testing, or formal verification to give users the extra assurance that their funds are safe.

### Time spent:
20 hours