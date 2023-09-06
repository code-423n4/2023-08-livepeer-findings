# QA report for Livepeer contest by Catellatech

## [01] Inconsistent Use of NatSpec
NatSpec is a boon to all Solidity developers. Not only does it provide a structure for developers to document their code within the code itself, it encourages the practice of documenting code. When future developers read code documented with NatSpec, they are able to increase their capacity to understand, upgrade, and fix code. Without code documented with NatSpec, that capacity is hindered.

The Livepeer codebase does have a high level of documentation with NatSpec. However there are numerous instances of functions missing NatSpec.

### Possible Risks
1. Lack of understanding of code without adequate documentation.
2. Difficulty in understanding, upgrading, and fixing code without documentation.

### Recommended Mitigation Steps
- Practice consistent use of NatSpec on all parts of the project codebase.
- Include the need for NatSpec comments for code reviews to pass.

## [02] Old version of OpenZeppelin Contracts used
Using old versions of 3rd-party libraries in the build process can introduce vulnerabilities to the protocol code.

- Latest is 4.9.3 (as of July 28, 2023), version used in project is 4.9.2

### Risks
- Older versions of OpenZeppelin contracts might not have fixes for known security vulnerabilities.
- Older versions might contain features or functionality that have been deprecated in recent versions.
- Newer versions often come with new features, optimizations, and improvements that are not available in older versions.

### Recommendation
Review the latest version of the OpenZeppelin contracts and upgrade.

## [03] Inconsistent coding style
An inconsistent coding style can be found throughout the codebase. Some examples include:
Some of the **parameter names** in certain functions lack the underscores as seen in other cases. To enhance code `readability` and adhere to the Solidity style guide standards, it is recommended to incorporate underscores in the parameter names. **This practice contributes to maintaining a consistent and organized codebase.**

In some functions, underscores are used: 
```solidity
SortedArrays.sol
28:  function findLowerBound(uint256[] storage _array, uint256 _val) internal view returns (uint256)

```
while in others they are not present:

```solidity
SortedArrays.sol
64: function pushSorted(uint256[] storage array, uint256 val) internal
```

To improve the overall consistency and readability of the codebase, consider adhering to a more consistent coding style by following a specific style guide.

## [04] Crucial information is missing on important functions in all contracts
In the constructor functions it is not specified in the documentation if the admin will be an EOA or a contract, for example in the Treasury contract. Consider improving the docstrings to reflect the exact intended behaviour, and using `Address.isContract` function from OpenZeppelin’s library to detect if an address is effectively a contract.

## [05] We suggest to use named parameters for mapping type
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18.
For example:

```Solidity 
mapping(address account => uint256 balance); 
```

