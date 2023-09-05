# Livepeer - Quality Assurance Report

# Low Risk Findings

## Table Of Content

| Number                                                                                                           | Issue                                                                                                | Instances |
| :--------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------- | --------: |
| [L-01](#l-01---contracts-should-use-the-latest-version-of-solidity-to-be-safe-from-bug-fixes)                    | Contracts should use the `latest version` of solidity to be safe from `bug fixes`                    |        10 |
| [L-02](#l-02---missing-validation-for-address0-in-the-constructors-or-in-the-functions-changing-state-variables) | Missing validation for `address(0)` in the constructors or in the functions changing state variables |         1 |
| [L-03](#l-03---shadow-variables)                                                                                 | Shadow Variables                                                                                     |         1 |

### [L-01] - Contracts should use the `latest version` of solidity to be safe from `bug fixes`.

**Details**

The contracts in scope use the older version of solidity (`0.8.9`) and the current version while writing this report is (`0.8.21`).

Solidity docs always suggest using the latest version of Solidity for the deployment of smart contracts.

There are no breaking changes between these versions but numerous minor bug fixes have been done between them.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

`Example`

```diff
-   pragma solidity 0.8.9;
+   pragma solidity 0.8.21;
```

`Findings Links`

[BondingManager.sol - Line 02](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L2)

[BondingVotes.sol - Line 02](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L2)

[IBondingManager.sol - Line 02](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L2)

[IBondingVotes.sol - Line 02](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingVotes.sol#L2)

[EarningsPoolLIP36.sol - Line 02](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L2)

[SortedArrays.sol - Line 02](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L2)

[GovernorCountingOverridable.sol - Line 02](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L2)

[IVotes.sol - Line 02](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/IVotes.sol#L2)

[LivepeerGovernor.sol - Line 02](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L2)

[Treasury.sol - Line 02](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/Treasury.sol#L2)

</details>

### [L-02] - Missing validation for `address(0)` in the constructors or in the functions changing state variables.

**Details**

It is recommended to check for `address(0)` when updating state variables through constructor or functions. This can save the protocol from accidental updates.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

`Example`

```solidity
require(
    _account != address(0) && _delegateAddress != address(0),
    "BondingVotes: Addresses should not be zero address"
);
```

`Findings Links`

[BondingVotes.sol - Line 259 - 265](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L259-L265)

</details>

### [L-03] - Shadow Variables.

**Details**

Consider the compiler warnings into consideration and avoid the variable shadowing.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

```diff
-   uint256 quota
+   uint256 quota_
-   __GovernorCountingOverridable_init(quota);
+   __GovernorCountingOverridable_init(quota_);
```

Here compilar is considering that `quota` is refering to the parent contract state variable and it can create issue.

[LivepeerGovernor.sol - Line 59](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L59)

</details>

# Informational Findings

## Table Of Content

| Number                                                                                                                                       | Issue                                                                                                                             | Instances |
| :------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------- | --------: |
| [I-01](#i-01---always-import-other-source-files-explicitly)                                                                                  | Always `import` other source files explicitly                                                                                     |         9 |
| [I-02](#i-02---mark-the-explicit-visibility-of-state-variables)                                                                              | Mark the explicit visibility of state variables                                                                                   |         1 |
| [I-03](#i-03---we-can-perform-the-simple-arthematic-operations-without-safemath-library-in-version-8)                                        | We can perform the simple arthematic operations without `SafeMath` library in version 8                                           |        8+ |
| [I-04](#i-04---function-parameter-can-be-silent-using---syntax)                                                                              | Function parameter can be silent using /\* \*/ syntax                                                                             |         1 |
| [I-05](#i-05---set-the-internal-layout-of-contracts-interfaces-and-libraries)                                                                | Set the Internal Layout of Contracts, Interfaces, and Libraries                                                                   |         5 |
| [I-06](#i-06---use-natspec-comments-for-events-state-variables-constructor-functions-and-all-other-things-which-will-be-included-in-the-abi) | Use `Natspec` comments for events, state variables, constructor, functions and all other things which will be included in the ABI |         7 |
| [I-07](#i-07---production-code-should-not-contain-the-todo-comments)                                                                         | Production code should not contain the TODO comments                                                                              |         1 |
| [I-08](#i-08---use-natspec-comments-for-smart-contracts-interfaces-and-libraries)                                                            | Use `Natspec` comments for smart contracts, interfaces and libraries                                                              |         2 |
| [I-09](#i-09---should-use-the-meaningful-error-messages-so-users-can-understand-easily)                                                      | Should use the meaningful error messages so users can understand easily                                                           |         1 |

### [I-01] - Always `import` other source files explicitly.

**Details**

Soldity docs suggest to always import explicitly because just import will import all of the things from the file.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

`Example`

```diff
-   import "../ManagerProxyTarget.sol";
+   import { ManagerProxyTarget } from "../ManagerProxyTarget.sol";
```

`Findings Links`

[BondingManager.sol - Line 04 - 17](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L4-L17)

[BondingVotes.sol - Line 04 - 14](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L4-L14)

[IBondingVotes.sol - Line 04](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingVotes.sol#L4)

[EarningsPoolLIP36.sol - Line 04 - 07](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L4-L7)

[SortedArrays.sol - Line 04 - 06](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L4-L6)

[GovernorCountingOverridable.sol - Line 04 - 14](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L4-L14)

[IVotes.sol - Line 04](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/IVotes.sol#L4)

[LivepeerGovernor.sol - Line 04 - 18](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L4-L18)

[Treasury.sol - Line 04](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/Treasury.sol#L4)

</details>

### [I-02] - Mark the explicit visibility of state variables.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

`Example`

```diff
-   uint256 constant MAX_FUTURE_ROUND = 2 ** 256 - 1;
+   uint256 internal constant MAX_FUTURE_ROUND = 2 ** 256 - 1;
```

`Findings Links`

[BondingManager.sol - Line 32](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L32)

</details>

### [I-03] - We can perform the simple arthematic operations without `SafeMath` library in version 8.

**Details**

Throughout the code `SafeMath` library is used for arthematic operations but after version `0.8.0` solidity itself take care of over/underflow.

The use of this library just increases the conversion operations cost.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

`Example`

```diff
-   delegators[msg.sender].fees = fees.sub(_amount);
+   delegators[msg.sender].fees = fees - _amount;
```

`Findings Links`

[BondingManager.sol - Line 282](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L282)

[BondingManager.sol - Line 322](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L322)

[BondingManager.sol - Line 349](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L349)

[BondingManager.sol - Line 356](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L356)

[BondingManager.sol - Line 364](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L364)

[BondingManager.sol - Line 369](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L369)

[BondingManager.sol - Line 377](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L377)

[BondingManager.sol - Line 892](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L892)

</details>

### [I-04] - Function parameter can be silent using /\* \*/ syntax.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

```solidity
function updateTranscoderWithFees(
    address _transcoder,
    uint256 _fees,
    uint256 _round
) external whenSystemNotPaused onlyTicketBroker {
    // Silence unused param compiler warning
    // _round;
```

here on [BondingManager.sol - Line 305](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L305) we are using the variable inside the function in order to silent it.

But solidity by default provide the feature for this functionality.

We can use this syntax to silent it.

```diff
function updateTranscoderWithFees(
    address _transcoder,
    uint256 _fees,
-    uint256 _round
+    uint256 /* _round */
) external whenSystemNotPaused onlyTicketBroker {
```

</details>

### [I-05] - Set the Internal Layout of Contracts, Interfaces, and Libraries.

**Details**

According to the Solidity [Style Guide](https://docs.soliditylang.org/en/v0.8.21/style-guide.html#style-guide) consistency in the projects layout is very important. If all projects apply the style guides provided by solidity docs then understanding the projects and its code will be much easier for developers, and auditors.

Solidity docs;

> A style guide is about consistency. Consistency with this style guide is important. Consistency within a project is more important. Consistency within one module or function is most important.

According to the solidity styles guide;

Inside each contract, library or interface, this order should be followed:

1. Type declarations
2. State variables
3. Events
4. Errors
5. Modifiers
6. Functions

And this should be the order of functions:

-   constructor
-   receive function (if exists)
-   fallback function (if exists)
-   external
-   public
-   internal
-   private
-   View and pure functions last

Apply the recommeded layout sturcture in all mentioned contracts according to solidity styles guide.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[BondingVotes.sol - Line 21 - 557](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol#L21-L557)

[IBondingVotes.sol - Line 10 - 52](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingVotes.sol#L10-L52)

[SortedArrays.sol - Line 13 - 80](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L13-L80)

[GovernorCountingOverridable.sol - Line 22 - 224](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol#L22-L224)

[LivepeerGovernor.sol - Line 43 - 167](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol#L43-L167)

</details>

### [I-06] - Use `Natspec` comments for events, state variables, constructor, functions and all other things which will be included in the ABI.

**Details**

It is recommended by solidity docs that solidity contracts should be fully annotated using NatSpec for all public interfaces (everything in the ABI).

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[IBondingManager.sol - Line 09 - 39](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L9-L39)

[IBondingManager.sol - Line 59 - 85](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L59-L85)

[IBondingVotes.sol - Line 10 - 16](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingVotes.sol#L10-L16)

[IBondingVotes.sol - Line 41](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingVotes.sol#L41)

[IBondingVotes.sol - Line 45 - 53](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingVotes.sol#L45-L53)

[IVotes.sol - Line 07 - 17](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/IVotes.sol#L7-L17)

[Treasury.sol - Line 16 - 23](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/Treasury.sol#L16-L23)

</details>

### [I-07] - Production code should not contain the TODO comments.

**Details**

The mentioned TODO is completed but the comment is still in the code which should be removed.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[IBondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/IBondingManager.sol#L6)

```diff
-   * TODO: switch to interface type
```

</details>

### [I-08] - Use `Natspec` comments for smart contracts, interfaces and libraries.

**Details**

It is recommended by solidity docs that solidity contracts should be fully annotated using NatSpec for all public interfaces (everything in the ABI).

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[EarningsPoolLIP36.sol - Line 09](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/EarningsPoolLIP36.sol#L9)

[IVotes.sol - Line 06](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/IVotes.sol#L6)

</details>

### [I-09] - Should use the meaningful error messages so users can understand easily.

**Details**

Normal users cannot understand the developers terms and meanings so should use the responses which users can easily understand. And adding the location trace also helps the developer to easily figure out the location of the issue.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[SortedArrays.sol - Line 72](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L71)

```diff
if (val < last) {
-    revert DecreasingValues(val, last);
+    SortedArrays__ValueMustBeGreaterThanLastArrayValue();
}
```

</details>