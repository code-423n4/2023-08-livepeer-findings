# Analysis - Livepeer Onchain Treasury Upgrade
# Summary

| List |Head |Details|
|:--|:----------------|:------|
|1 | Overview of Livepeer  | overview of the key components and features of Livepeer |
|2 |Audit approach | Process and steps i followed  |
|3 |Learnings | Learnings from this protocol|
|4 |Possible Systemic Risks | The possible systemic risks based on my analysis |
|6 | Centralization risks | Concerns associated with centralized systems |
|5 |Code Commentary and Risks | Suggestions for existing code base and risks based on codebase |
|7 |Gas Optimizations | Details about my gas optimizations findings and gas savings  |
|9 |Non-functional aspects | General suggestions |
|10 |Time spent on analysis  | The Over all time spend for this reports |

## Overview
> Decentralized ``video infrastructure protocol`` powering video in ``web3's`` leading ``social`` and ``media`` applications

``Livepeer`` is a protocol for developers who want to add ``live`` or ``on-demand video`` to their project. It aims to increase the ``reliability`` of video streaming while ``reducing costs`` associated with it by up to ``50x``.

To achieve this ``Livepeer`` is building ``p2p`` infrastructure that interacts through a ``marketplace ``secured by the ``Ethereum blockchain``.

### Who Can Benefit from Livepeer

#### Developers
Who want to build ``applications`` that include ``live`` or on ``demand video`` can use ``Livepeer`` to power their ``video functionality``

#### Users
Who want to ``stream video``, ``gaming``, ``coding``, ``entertainment``, ``educational courses``, and other types of content can use applications built on ``Livepeer`` to do so.

#### Broadcasters
Such as Twitch who have ``large audiences`` and ``high streaming bills`` or ``infrastructure costs`` can use ``Livepeer`` to reduce costs or ``infrastructure overhead``.

### How Livepeer Works ?
When ``Bob`` opens the app and taps ``Record`` at the start of each game, the app sends the ``live video`` along with ``broadcaster fees`` into the ``Livepeer network``. ``Livepeer`` then ``transcodes`` the ``video`` into ``all the formats`` and ``bitrates`` that his ``viewers`` can consume.

###  Key actors in Livepeer network

  - Orchestrators  
  - Delegators

### ORCHESTRATORS
- In ``Livepeer``, anyone can join the ``networ``k and become what's known as an ``orchestrator`` by running software that allows you to ``contribute`` your computer's resources ``(CPU, GPU, and bandwidth)`` in service of ``transcoding`` and ``distributing video`` for paying ``broadcasters`` and ``developers`` 

- For doing so, you ``earn fees`` in the form of a ``cryptocurrency`` like ``ETH`` or a ``stablecoin`` pegged to the ``US dollar`` like ``DAI``

- In order to earn the right to do this type of work on the ``network``, you must first ``earn`` or ``acquire Livepeer Token``, also known as ``LPT``

### DELEGATORS

- ``Delegators`` are ``Livepeer tokenholders`` who participate in the ``networ``k by ``staking`` their tokens towards ``orchestrators`` who they believe are doing good and honest work.

#### LPT Token (Livepeer Token)

The purpose of the ``Livepeer token (LPT)`` is to ``coordinate``, ``bootstrap``, and ``incentivize participants`` to make sure the ``Livepeer network`` is as ``cheap``, ``effective``, ``secure``, ``reliable`` and ``useful as possible``.

#### Rewards Distrubution over ORCHESTRATORS and DELEGATORS

- When a ``broadcaster`` pays fees into the ``network``, both ``orchestrators`` and ``Delegators`` earn a portion of those ``fees`` as a reward for ensuring a ``high-quality`` and ``secure network``.

- In addition to ``earning fees``, ``Livepeer`` mints ``new token`` over time, much like ``Bitcoin`` and ``Ethereum`` block rewards, which are ``split`` amongst ``Delegators`` and ``orchestrators`` in proportion to their ``total stake`` relative to others in the network.

 
## Audit approach

I followed below steps while analyzing and auditing the code base.

1. Initial Read and Note-Taking
 Reading the contest's Readme.md file and took essential notes

  - Livepeer Onchain Treasury Upgrade
    - Inheritance
    - ``100%`` Test Suit
    - This an ``upgrade`` of an ``existing system`` - checkpointing support for an ``existing staking system``, abstract it as an ``IERC5805`` token and set up an ``OpenZeppelin Governor``
    - Uses ``Timelock`` and ``OZ``
    - Deployed on ``L2`` but ``no bridges`` involved
    - Protocol is already audited 
    - Areas to be addressed
      - Find any inconsistency in the stake checkpoint with the actual stake in BondingManager in the corresponding round (as per the LIP-89 spec);
      - Force the vote counting logic to behave differently from the specification, fabricating or losing voting power, altering other users votes (apart from the override), etc;
      - Exploit the governor/treasury into running arbitrary code without a proper proposal;
      - Find any loopholes for the LivepeerGovernor to make changes to the protocol itself;
      - Distort the treasury contribution to something different than what is configured;
      
2. Analyzed the over all codebase one iterations very fast

3. Study of documentation to understand each contract purposes, its functionality, how it is connected with other contracts, etc.

4. Then i read old audits and already known findings. Then go through the bot races findings 

5. Then setup my testing environment things. Used yarn commands to test the protocol 

#### Commands Used for testing: 

  - yarn compile
  - yarn test:audit

6. Finally, I started with the auditing the code base in ``depth`` way I started understanding line by line code and took the ``necessary notes`` to ask some questions to ``sponsors``.

## Stages of audit

- ``The first stage of the audit``

During the ``initial stage`` of the audit for ``Livepeer Onchain Treasury Upgrade``, the primary focus was on analyzing ``GAS`` usage and ``Quality assurance (QA)`` aspects. This phase of the audit aimed to ensure the efficiency of gas consumption and verify the robustness of the platform.

- ``The second stage of the audit``

In the ``second stage`` of the audit for ``Livepeer Onchain Treasury Upgrade``, the focus shifted towards understanding the ``protocol`` usage in more detail. This involved identifying and analyzing the ``important contracts`` and functions within the system. By examining these key components, the audit aimed to gain a comprehensive understanding of the protocol's ``functionality`` and ``potential risks``. 

- ``The third stage of the audit``

During the ``third stage`` of the audit for ``Livepeer Onchain Treasury Upgrade``, the focus was on thoroughly examining and marking any doubtful or vulnerable areas within the protocol. This stage involved conducting comprehensive ``vulnerability assessments ``and identifying ``potential weaknesses`` in the system. Found ``60-70`` ``vulnerable`` and ``weakness`` code parts all marked with ``@audit tags``.

- ``The fourth stage of the audit``

During the ``fourth stage`` of the audit for ``Livepeer Onchain Treasury Upgrade``, a ``comprehensive analysis`` and ``testing`` of the previously identified ``doubtful`` and ``vulnerable`` areas were conducted. This stage involved diving deeper into these areas, performing ``in-depth examinations``, and subjecting them to ``rigorous testing``, including ``fuzzing`` with various inputs. Finally concluded findings after all ``research's`` and ``tests``. Then i reported ``C4`` with ``proper formats`` 

## Key Learnings 

``Livepeer`` is a ``blockchain-based protocol`` tailored for ``developers`` seeking to ``integrate live`` or ``on-demand video`` streaming into their applications. It ``leverages`` a ``decentralized marketplac``e secured by the ``Ethereum blockchain`` to enhance the ``reliability`` of ``video streaming`` while ``dramatically`` reducing ``associated costs``, potentially up to ``50 times``. Within this ``ecosystem``, ``Orchestrators`` play a ``pivotal`` role by contributing ``computing resources`` (CPU, GPU, and bandwidth) to ``transcode`` and distribute video content for paying ``developers`` and ``broadcasters``. They receive compensation in the form of ``cryptocurrenc``y, such as ``ETH`` or stablecoins pegged to the US dollar (e.g., DAI). Additionally, the ``Livepeer token (LPT)`` serves as a critical ``incentive mechanism``, allowing token holders to perform transcoding work on the network in ``exchange for fees``. ``Delegators``, another crucial ``participant group``, secure the ``network`` by ``staking their LPT tokens`` with ``Orchestrators`` they trust. This system ensures ``high-quality`` and secure video streaming experiences. ``Orchestrators`` and ``Delegators``, through their active participation, also ``benefit ``from token ``rewards`` and network ``ownership growth``. The protocol operates in ``rounds``, with each round lasting approximately ``22.4 hours``, and the ``inflation rate`` automatically adjusts based on ``token staking``, further incentivizing participation to ``maintain network health``.

## Possible Systemic Risks

#### Network Congestion and Scalability
High ``congestio``n could ``disrupt`` Livepeer's video ``transcoding ``and ``streaming operations``, potentially resulting in reduced ``quality of service`` and ``higher costs``

#### Orchestrator Reliability
 ``Inconsistent Orchestrator`` performance can lead to ``service interruptions`` and affect the overall quality and ``reliability`` of ``Livepeer's video streaming capabilities``
 
#### Token Volatility
Significant fluctuations in ``LPT'``s value can ``influence`` the economic incentives for ``network participants``, potentially affecting the ``stability ``and ``security of the Livepeer ecosystem``

#### Lack of Decentralization
``Over-centralization``, where a small number of ``Orchestrators`` dominate the network, may ``compromise Livepeer's intended decentralization and security``

## Centralization risks

Contracts with privileged functions that rely on a ``single owner`` or ``admin`` for control can introduce several risks

```solidity
FILE: 2023-08-livepeer/contracts/bonding/BondingManager.sol

155:     function setUnbondingPeriod(uint64 _unbondingPeriod) external onlyControllerOwner {

167:     function setTreasuryRewardCutRate(uint256 _cutRate) external onlyControllerOwner {

176:     function setTreasuryBalanceCeiling(uint256 _ceiling) external onlyControllerOwner {

186:     function setNumActiveTranscoders(uint256 _numActiveTranscoders) external onlyControllerOwner {

```
- setUnbondingPeriod ()
The ability to modify the ``unbonding period`` is concentrated in the hands of the ``controller owner``. This means that ``participants`` must trust this ``single entity`` to make responsible decisions regarding the ``unbonding period``. If the controller owner were to act ``maliciously`` or make ``unfavorable changes``, it could disrupt the protocol and ``potentially harm users``.

- setTreasuryRewardCutRate()
Centralizing control over this parameter means that the controller ``owner`` has significant influence over the protocol's ``economic incentives``. Users have no direct say in determining the ``cut rate``, which could affect their ``rewards``. Trust is placed in the controller ``owner`` to act in the best interests of the network.

- setTreasuryBalanceCeiling()
Centralized control over the ``treasury`` balance ceiling means that decisions regarding the ``maximum amount`` of funds the protocol can hold are made solely by the controller ``owner``. If this limit is set ``too high`` or ``too low``, it can have significant ``financial implications`` for the ``protocol``

- setNumActiveTranscoders()
The number of active ``transcoders`` directly affects the ``network's performance`` and ``decentralization``. Centralized control over this parameter means that the controller ``owner`` can influence the protocol's ``centralization`` level without the need for ``community consensus``

## Gas Optimizations

The ``Livepeer protocol`` demonstrates several gas optimization opportunities that could ``enhance`` the ``efficiency`` of its smart contracts and ``reduce transaction costs``. First, the use of the ``memory`` keyword instead of ``storage`` when accessing ``variables`` for the ``second time`` has been identified as a cost-saving measure.This optimization has been observed in the context of ``lock.withdrawRound`` and similar variables within the ``protocol``. Furthermore, optimizing ``struct`` packing by using ``uint96`` instead of ``uint256`` for variables like ``lastClaimRound`` in certain data structures offers ``additional savings``. In addition, gas-efficient programming practices recommend placing ``require`` statements that check input arguments at the ``beginning of functions``, allowing for ``early and less costly failure``s. Lastly, ``caching state`` variables used multiple times within a function in ``stack variables``, such as ``withdrawRound``, ``bondedAmount``, and ``delegateAddress``, can lead to significant savings in terms of both ``GAS`` and ``SLOD ``operations

## Code Commentry and Risks 

[BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol)

### Code Commentry

- In the ``constructor``, consider adding ``input validation`` to ensure that the provided ``_controller`` address is not zero ``(address(0))``. This will prevent ``accidental deployment`` with an ``invalid controller address``
- ``setUnbondingPeriod``, ``setTreasuryBalanceCeiling``, ``setNumActiveTranscoders``, ``setTreasuryRewardCutRate``,``setTreasuryRewardCutRate``  and other similar functions, consider adding ``validation checks`` to ensure that the input parameters are within ``acceptable ranges``. For example, ensure that ``_unbondingPeriod`` ``> 0``, ``> MIN`` and ``< MAX``
- For ``MAX_FUTURE_ROUND``, consider adding a ``brief explanation`` of why this value is set to ``2**256 - 1``. This can help other ``developers`` understand the ``purpose of this constant``
- Add visibility modifers for ``MAX_FUTURE_ROUND`` constant
- Implement ``checks-effects-interactions`` pattern in functions that interact with ``external contracts``. Ensure that ``state changes`` occur after external calls to prevent ``reentrancy vulnerabilities``
- In the ``TranscoderUpdate`` event, you could include the previous values of ``rewardCut`` and ``feeShare``.
- Consider renaming ``_cutRate`` in the ``setTreasuryRewardCutRate`` function to something like `` _newCutRate`` for clarity
- Maintain a consistent ``naming convention`` for ``variables`` and ``functions``. For example, stick with ``camelCase`` also private varibales should start with ``_`` as per solidity naming standard
- In the ``bondForWithHint`` and ``unbondWithHint`` functions, consider ``consolidating the logic`` for changing the ``delegate`` and updating the ``bonded amount`` into a separate internal function to enhance ``readability`` and reduce ``code duplication``
- Ensure that all relevant ``events`` are emitted consistently when ``state changes`` occur. This allows ``external applications`` to listen for and react to these events
-  Implement ``fallback`` and ``receive`` functions for accidential ether send to contract to avoid unexpected behavior
-  Add emergency stop functions and functions to recover accidential ether send
-  Consider implementing ``access control`` mechanisms to restrict certain functions to authorized users or roles. ``OpenZeppelin's Access Control`` library is a useful tool for this purpose
-  

### Risks

1. The contract relies on an "owner" (Controller owner) to make critical parameter updates, such as ``setUnbondingPeriod``, ``setTreasuryRewardCutRate``, ``setTreasuryBalanceCeiling``, and ``setNumActiveTranscoders``. If the ``owner's address`` is ``compromised`` or ``acts maliciously``, it could lead to ``inappropriate`` changes in ``protocol parameters``
2. The contract uses a loop to update ``transcoders' states`` in the ``reward function``. While the loop appears bounded by the ``number of transcoders``, exceeding gas limits can lead to function ``execution failures``
3. ``Large stakeholders`` or ``transcoders`` with significant ``stake`` can potentially ``manipulate the protocol's rewards`` or ``governance decisions`` 
4. In functions like ``bondForWithHint`` and ``transferBond``, it's important to consider the ``orde``r in which ``transactions are processed``, especially when it involves changing ``delegate addresses`` or ``transcoder positions`` in the pool. Transaction ordering can impact the final state of the contract, potentially leading to ``unintended behavior`` 
5. If multiple ``users`` or ``contracts`` interact with your ``contract`` concurrently, ``race conditions`` might arise.
6. Be aware of the implications of adding or removing transcoders from the pool. This can affect the selection of active transcoders and their rewards. Ensure that the pool remains in a consistent state after each operation

[BondingVotes.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingVotes.sol)

### Code Commentry

- Import only needed things instead of over all contracts 
- Instead of using ``if`` statements for validation and reverting, consider using the ``require`` statement. This makes the code more explicit and easier to read.
- In several places, you check if`` _round`` is greater than or equal to ``clock() + 1``. You can consolidate this check into a ``helper function`` to reduce ``code repetition`` and make it ``more readable``
- In the ``bondingManager`` and ``roundsManager`` functions, you can use a ``bytes32 constant`` to represent the contract name for ``getContract``. This can make the code more ``readable`` and ``consistent``
- Replace ``magic numbers`` with ``named constants`` to improve ``code readability`` and ``maintainability``

### Risks

1. The ``clock()`` function relies on ``block timestamps`` to determine the ``current round``. Any ``manipulation`` of ``block timestamps`` can affect the accuracy of ``round calculations``.
2. Functions with ``complex calculations`` and storage operations, such as ``delegatorCumulativeStakeAt``, may be at risk of exceeding the ``gas limit`` for ``Ethereum transactions``
3. This risk could manifest in the ``getBondingCheckpointAt`` and ``delegatorCumulativeStakeAt`` functions, which retrieve and ``manipulate bonding`` checkpoint data. If these functions do not accurately reflect the ``state of the system``, ``data consistenc``y issues may arise
4. The code uses ``sorting operations`` in functions like ``getTotalActiveStakeAt`` and ``getBondingCheckpointAt``. Sorting algorithm used is ``inefficient``, it can lead to ``higher gas`` costs when querying ``historical data``. Consider ``optimizing sorting algorithms`` 

[GovernorCountingOverridable.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/GovernorCountingOverridable.sol)

### Code Commentry

- The code imports several contracts and interfaces from OpenZeppelin, such as ``Initializable``, ``GovernorUpgradeable``, and IERC5805Upgradeable. It also imports contracts and libraries ``specific to the project``
- Add ``comprehensive comments`` and documentation throughout your code. Explain the purpose of ``functions``, ``variables``, and ``complex logic``. Good documentation makes it easier for other developers (and your future self) to understand and maintain the code
- Consider using function ``modifiers`` to enhance ``code readability`` and ``maintainability``
- The code defines custom vote types (VoteType) to represent "For," "Against," and "Abstain" votes, aligning with the Governor Bravo's ordering. This helps maintain clarity in the code and the voting process

### Risks

1. f the votes() contract is a delegate call to an external contract, there are potential risks associated with delegate calls, such as reentrancy vulnerabilities. Carefully audit the external contract and consider using the latest best practices for secure delegate calls
2. The code imports contracts and libraries from external sources, such as OpenZeppelin. Ensure that these dependencies are up to date and compatible with the specified Solidity version to avoid vulnerabilities from outdated code
3. If this contract is part of a larger system, consider the implications of contract upgrades on governance and voting. Upgrades should not compromise the integrity of past votes or introduce unexpected changes in governance rules
4. This risk could potentially affect the entire contract, especially where timestamps are used for vote delegation and tracking. The relevant code areas include functions like proposalSnapshot, where timestamps are used to determine delegation eligibility
5. The gas cost for tallies can be a concern in functions that tally votes, such as _quorumReached and _voteSucceeded. If there are a large number of proposals or voters, gas costs could accumulate in these functions

[LivepeerGovernor.sol](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/treasury/LivepeerGovernor.sol)

### Code Commentry

- Try to reduce the inheritance list
- _controller not checked with address(0)
- initialVotingDelay,initialVotingPeriod,initialProposalThreshold,initialQuorum,quota Lack of checkes for interger values 
- IVotes(controller.getContract(keccak256("BondingVotes"))) , Treasury(payable(controller.getContract(keccak256("Treasury")))) save this with constants
- Array length not checked in _execute(),_cancel() functions
- consider adding comments to internal functions as well, especially if they contain complex logic. This helps future developers understand the purpose and behavior of these functions
- While Solidity automatically assigns the internal visibility to functions that don't specify visibility, consider explicitly specifying visibility to improve code readability and clarit

## Non-functional aspects

- Aim for high test coverage to validate the contract's behavior and catch potential bugs or vulnerabilities
- The protocol execution flow is not explained in efficient way. 
- Its best to explain over all protocol with architecture is easy to understandings
- Consider Using upgradable contracts  

## Time spent on analysis 
``15 Hours``

































































### Time spent:
15 hours