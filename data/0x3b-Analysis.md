# Summary
The codebase demonstrates a solid foundation with intriguing concepts, although there is room for improvement. Various mechanisms were reviewed, including governance in [GovernorCountingOverridable.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol), arrays in [SortedArrays.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol), and, most importantly, bonds in [BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol). Each of these components offers unique features and functionalities. Recommendations have emphasized that while the current system is comprehensive and sophisticated, it suffers from excessive complexity. A balanced and refined approach, aimed at reducing this complexity, is essential for a well-functioning and secure system.

| Severity | Occurrences |
|----------|-------------|
| High     | 0           |
| Medium   | 1           |
| Low      | 2           |
| Gas      | 2           |

# Time allocations

|            |            |
|------------|------------|
| Start date | 01.09.2023 |
| End date   | 04.09.2023 |
| Total      | 4 days    |

| Members       | Positions            | Time spent |
|---------------|----------------------|------------|
| **0x3b**      | full time researcher | 30H+       |



# Findings
Only one medium finding was submitted, and although it was unfortunate to find only one vulnerability, it actually demonstrates the proficiency of the developers. Additionally, suggestions for enhancements, optimizations, and improvements were also included, albeit not many. Having dedicated ample time to thoroughly examine the code, I delved deeply into various concepts, striving to uncover as many vulnerabilities as possible. Unfortunately, I did not find many, as the code was too complex for quick comprehension, with extensive unit tests covering every possible scenario. Every scenario I attempted had already been tested and fixed.


# Codebase quality
The code quality demonstrates a solid foundation, featuring intriguing concepts alongside conventional mechanics. However, there's room for refinement to further elevate its performance. The main issue with this system is its introduction of unnecessary complexity. While it hasn't yet led to many vulnerabilities, it's like a spider web that starts small, but can rapidly expand and entangle the spider into his own creation.

# Mechanism review

###### BondingManager.sol
- The largest and most complex contract in the system.
- It offers the main functionality.
- Users use it to bond and delegate.
- Transcoders use it to manage their balances.
- It has a mechanism to claim fees and rewards.

###### BondingVotes.sol
- A helper contract connected to **BondingManager**.
- Used to reduce complexity in **BondingManager**.
- The main functionality is to checkpoint transcoders.

###### EarningsPoolLIP36.sol
- A contract that manages the earnings of the transcoders.
- It can assign the fees that transcoders take.
- Used to reduce complexity in **BondingManager**.

###### SortedArrays.sol
- A contract that manages arrays.
- It has an interesting function, [findLowerBound](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/libraries/SortedArrays.sol#L28-L55).
- Manages sorted arrays.

###### GovernorCountingOverridable.sol
- The main governor contract that extends the OZ implementation.
- It includes a mechanism for counting votes and quota.
- Sophisticated checks that consider the percentage of "for" votes, instead of a raw ( > ) comparison between "against" and "for."
- Interesting concepts of delegators being able to override the delegatee's votes.

###### Treasury.sol
- OZ timelock implementation.

###### LivepeerGovernor.sol
- The main governor contract.
- Implements all the small parts of governance.
- Combines **Treasury** and **GovernorCountingOverridable**.


# Architecture recommendations
Livepeer's existing framework is quite comprehensive, encompassing advanced bonds and nuanced governance implementations that empower delegators to override delegatee votes. Within the parameters of the current scope, the system appears to be well-equipped. It's worth noting that in Solidity, additions don't necessarily equate to improvements. Instead, additional complexity can introduce higher risks in a non-linear, exponential manner. Therefore, maintaining a balanced and refined approach is key to a stable system.

# Centralization risk
Considering the provided scope, Livepeer demonstrates an exceptionally high level of decentralization. I am truly fascinated (If you can't tell by now xD) by the way delegators can override delegatee decisions. This empowerment further amplifies the influence of ordinary users, enabling the system to achieve even greater decentralization. In contrast to other systems where you cannot extract any value (such as rewards or fees) without becoming a delegator, and once you delegate, you forfeit the ability to make different decisions than the person you've delegated to. This decentralization paves the way for a brighter future where all users are empowered to shape the ecosystem as they see fit. Unfortunately, despite these benefits, decentralization also brings a degree of slowness in adaptation, and in the crypto world, the entire landscape can change dramatically within a few months. This scenario favors only the swift adapters while leaving the slower ones behind.

# Systemic risks
Since this is their first review, it's normal to find some bugs, although not many and small in damage. There's still room for improvement.I might have found only one existing issue,however I am quite sure other participants will discover a few more vulnerabilities.

For systems as complex as this one, it's a good idea to consider conducting another review after this initial one. This second review could help uncover any significant problems that might have been missed. Complicated systems often require multiple audits to ensure they're safe and functioning effectively.

### Time spent:
30 hours