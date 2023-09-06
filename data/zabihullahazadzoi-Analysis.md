# Livepeer Onchain Treasury Upgrade - Analysis report

|   Heading   |  Details   |
|------|----------|
|The methodology employed to assess the codebase|The strategies used to analyze and appraise the protocol.|
|Codebase quality analysis|Its structure, readability, maintainability, and adherence to best practices|
|Centralization risks|Power, control, or decision-making authority is concentrated in a single entity|
|Gas Optimization|Process to reduce the amount of gas consumed on a blockchain network|
|Other recommendations|Recommendations for improving the quality of your codebase|
|Time spent on analysis|Time detail spent in code review and analysis|


# The methodology employed to assess the codebase

1. **Understand the Project Requirements and Objectives**: Begin by understanding the purpose and objectives of Dopex Protocol. What is it designed to do, and what are the expected outcomes?, once you have a fundamental understanding of the overall protocol you can start to scope the analysis. This involves identifying the specific areas of code that you want to focus on.

2. **Review Documentation**: Check if there is any documentation accompanying the code. This can include a whitepaper, technical documentation, or comments within the code itself. Understanding the context will help in review.

3. **Code Quality Tools**: Utilize code quality tools and static analysis tools like Slither to automatically identify potential issues icluding insecure coding practices and common vulnerabilities.

4. **Manual Code Review**: Conduct a detailed manual review of the codebase. Pay attention to:

	* Code structure and organization
	* Variables and data types
	* Security vulnerabilities
	* External dependencies
	* Access Control
	* Gas efficiency
	
5. **Testing**: After a comprehensive review of the codebase, you should perform a series of tests to ensure that it works as intended. This should include:
	* Unit tests to verify individual functions and components.
	* Integration tests to check how different parts of the contract interact.
	* Fuzz testing to identify vulnerabilities by providing unexpected inputs.
	* Scenario testing to simulate real-world use cases.
	
	
# Codebase quality analysis

Overview of the Codebase:

### `EarningsPoolLIP36.sol`:
* This contract is a Solidity library named EarningsPoolLIP36 that provides functions to manage and calculate earnings in a staking pool. It uses the SafeMath library from OpenZeppelin for safe arithmetic operations and a custom library called PreciseMathUtils for precise mathematical operations.
* The contract assumes that the data provided by the EarningsPool contract is accurate and reliable. If the EarningsPool contract has any issues, it could affect this contract.
* The contract does not seem to validate the inputs to its functions. This could potentially lead to unexpected behavior if incorrect inputs are provided.

### `SortedArrays.sol`: 
* This contract is a Solidity library named SortedArrays that provides two main functionalities for handling sorted arrays of uint256:
	- finding the last element of an array that is lower or equal to an input value, using `findLowerBound` funtion.
	- pushing a value into a sorted array that is greater than or equal to the last element in the array, using `pushSorted` function.
* The pushSorted function relies on the user to maintain the sorted order of the array. If the user pushes values in the wrong order, the function will revert the transaction. This could be error-prone and might lead to unexpected reverts.
* The contract does not provide a function to sort an array. It assumes that the array is already sorted. If the array is not sorted, the findLowerBound function might not work correctly.
* Consider implementing a function that sorts the array automatically. This would make the library more user-friendly and reduce the risk of errors when users push values in the wrong order.

### `BondingManager.sol`: 
* The primary purpose of this contract is to facilitate the operation of transcoders and delegators within a video streaming network. Transcoders perform video transcoding tasks and earn rewards, while delegators bond their tokens to transcoders and also earn a portion of the rewards.
* There are mathematical operations on percentages and cumulative factors. Precision issues in these calculations can lead to incorrect distribution of rewards and fees.
* Some functions, especially those related to earnings calculations, can consume a significant amount of gas, making them costly to execute.
* Depending on the number of transcoders and delegators in the system, the contract's storage costs can become substantial, potentially leading to scalability issues.
* The contract's complexity might make it challenging to audit and ensure its security comprehensively. A thorough audit is crucial.
* The contract does not appear to have explicit checks against reentrancy attacks, which can be a security concern.
* The contract relies on other contracts like RoundsManager and BondingVotes. Any vulnerabilities in these dependencies could impact this contract's functionality.

### `BondingVotes.sol`:
* This contract is used for managing the voting power of delegators and transcoders in the network. 
* The contract does not use the Checks-Effects-Interactions pattern in all its external functions, which could potentially make it vulnerable to reentrancy attacks.
* This contract uses block timestamps (through the roundsManager().currentRound() function), which can be manipulated by miners to some degree.
* BondingVotes contract interacts with external contracts (like BondingManager and RoundsManager). If these contracts are compromised or behave unexpectedly, it could affect this contract as well.

### `GovernorCountingOverridable.sol`: 
* This contract, named GovernorCountingOverridable, is a governance contract that allows for voting on proposals and is built on top of OpenZeppelin's GovernorUpgradeable contract.
* The contract is quite complex, especially with the vote override functionality. This could potentially lead to bugs or unexpected behavior.
* The contract includes a storage gap for future variables. This is good for upgradeability, but it also means that the contract's storage layout is not fully determined, which could potentially lead to bugs or issues in future versions.
* The contract uses custom error types, which can make it harder to understand the contract's behavior.

### `LivepeerGoverner.sol`: 
* LivepeerGovernor, is a governance contract for the Livepeer protocol. It uses OpenZeppelin's upgradeable contracts and governance extensions for provide a governance system.
* The contract inherits from several OpenZeppelin contracts, including GovernorUpgradeable, GovernorSettingsUpgradeable, GovernorTimelockControlUpgradeable, GovernorVotesUpgradeable, GovernorVotesQuorumFractionUpgradeable, and GovernorCountingOverridable. This provides the contract with a range of governance-related functionalities, such as proposal creation, voting, and execution of approved proposals.
* The contract uses the initializer modifier from OpenZeppelin to ensure that initialization functions can only be called once. This prevents potential re-entrancy attacks.
* The contract relies heavily on the Controller contract. If the Controller contract is compromised or controlled by a malicious actor, it could affect the governance process and the behavior of this contract as well.

### `Treasury.sol`: 
* This a simple contract that is designed to hold and execute proposals for the LivepeerGovernor.
* This contract uses the Initializable contract, which provides a basic mechanism to avoid calling an initializer function more than once.
* The contract does not have any comments explaining what the initialize function parameters should be. This could make the contract harder to use correctly.


# Centralization risks

The codebase overall does not appear to have any explicit severe and critical centralizaion risks,
but still there are some points that should be considered:
- In the `BondingVotes` contract: 
	* Some functions are restricted to only be callable by the BondingManager contract. If the BondingManager contract is compromised, it could lead to unauthorized actions.
	* The contract inherits from ManagerProxyTarget, which suggests that it might be part of an upgradable contract system. If the governance process that controls upgrades is centralized, it could be a risk.
	* The contract is dependent on external contracts like BondingManager and RoundsManager. If these contracts are controlled by a centralized
entity, it could pose a risk.
		
- In `LivepeerGovernor` contract, The contract does have some centralization risks, primarily due to its reliance on the Controller contract. The Controller contract is used to fetch the addresses of the BondingVotes and Treasury contracts. If the Controller contract is compromised or controlled by a malicious actor, it could manipulate these addresses, which could disrupt the governance process.
Additionally, the initialize function, which sets up the initial state of the contract, can only be called once and does not appear to have any access controls. This means that whoever deploys the contract has a significant amount of control over its initial configuration.
These risks can be mitigated through careful access control, rigorous security practices, and transparent governance processes.
	
- In `Treasury` contract, the contract itself does not inherently introduce centralization risks, but the way it is used could potentially do so.
The initialize function takes an admin address as a parameter, which is used to set the initial admin of the TimelockController. If this admin address is a single centralized entity, then that entity has the power to propose, queue, and execute transactions, which is a centralization risk.
However, if the admin address is a decentralized autonomous organization (DAO) or a multisig wallet, then the centralization risk is mitigated.
It's also worth noting that the initialize function can only be called once due to the initializer modifier, which prevents the admin from being changed after initialization. This could be a centralization risk if the initial admin is a single entity, as they would have permanent control over the contract.
	

# Gas Optimization
* consider removing unused internal function.
* Reconsider assinging model for structs: consider altering the method used to assign values to data structures particularly structs, as this can lead to significant gas savings.
* Remove the initializer modifier: If we can just ensure that the initialize() function could only be called from within the constructor, we shouldn’t need to worry about it getting called again.
* consider inlining the modifiers that are used onece: y inlining a modifier that is used only once and not being inherited, you can eliminate the overhead of the generated code and reduce the gas cost of your contract.
* Use assembly to perform efficient back-to-back function calls.
	

# Other recommendations

* since some contracts lack events, try to emit events to track important state changes and function calls. This can make it easier to monitor the contract's activity and debug issues.
* Consider adding an emergency pause function that can halt certain contract operations in the event of a detected issue or attack. This can provide time to diagnose and fix the issue without risking further harm.
* Be cautious about interactions with external contracts, such as the `voting power provider`. Ensure that these contracts are secure and do not introduce centralization risk.
* Ensure that the upgrade process for the contracts is well-managed and transparent to prevent centralization risk. This could include a multi-signature wallet, a time lock, or a decentralized autonomous organization (DAO) to approve upgrades.
* Consider using standard error messages instead of custom error types to make it easier for others to understand the contract's behavior.
* Consider adding reentrancy guards to functions that make external calls to prevent potential reentrancy attacks.


# Time spent on analysis

* Day 1: Going through architecture of protocol and codebase review
* Day 2: Finish up the analysis and write the report


### Time spent:
23 hours