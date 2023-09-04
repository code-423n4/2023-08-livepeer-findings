**General**

The Livepeer Onchain Treasury Upgrade is a significant upgrade to the Livepeer protocol that introduces an on-chain treasury for the protocol. The treasury will be funded by a percentage of the protocol rewards, and it will be used to support the development and maintenance of the protocol.

The upgrade is designed to be secure and robust, and it has been audited by a number of independent security experts. However, there are still some potential risks associated with the upgrade, which are discussed in more detail below.

**BondingManager**

The BondingManager contract is the most critical contract in the Livepeer protocol, as it manages the protocol bonds (aka staking). The upgrade to the BondingManager contract introduces a number of new features, including:

* Support for treasury rewards minting
* Checkpointing of the BondingManager state

The treasury rewards minting feature is designed to ensure that the treasury is properly funded. The checkpointing feature is designed to ensure that the BondingManager state is always consistent, even in the event of a hard fork.

The upgrade to the BondingManager contract has been carefully designed to be secure and robust. However, there are still some potential risks associated with the upgrade, such as:

* The treasury rewards minting feature could be exploited to mint LPT tokens to an unauthorized address.
* The checkpointing feature could be exploited to corrupt the BondingManager state.

**BondingVotes**

The BondingVotes contract is a new contract that is introduced by the Livepeer Onchain Treasury Upgrade. The BondingVotes contract is responsible for managing the voting power of bonded users.

The BondingVotes contract uses the LIP-36 staking rewards calculation algorithm to calculate the voting power of a bonded user. The LIP-36 staking rewards calculation algorithm is a secure and robust algorithm that has been audited by a number of independent security experts.

The upgrade to the BondingVotes contract has been carefully designed to be secure and robust. However, there are still some potential risks associated with the upgrade, such as:

* The BondingVotes contract could be exploited to manipulate the voting power of bonded users.
* The BondingVotes contract could be exploited to censor votes from certain users.

**GovernorCountingOverridable**

The GovernorCountingOverridable contract is an abstract contract that is used by the LivepeerGovernor contract. The GovernorCountingOverridable contract provides a number of features that are useful for implementing a governance system, such as:

* Support for delegators to override their delegates' votes
* Support for a treasury balance ceiling

The upgrade to the GovernorCountingOverridable contract has been carefully designed to be secure and robust. However, there are still some potential risks associated with the upgrade, such as:

* The GovernorCountingOverridable contract could be exploited to manipulate the voting power of delegators.
* The GovernorCountingOverridable contract could be exploited to drain the treasury.

**Treasury**

The Treasury contract is a new contract that is introduced by the Livepeer Onchain Treasury Upgrade. The Treasury contract is responsible for managing the funds in the treasury.

The Treasury contract is a simple contract that does not have any complex logic. However, there are still some potential risks associated with the Treasury contract, such as:

* The Treasury contract could be exploited to drain the treasury.
* The Treasury contract could be exploited to mint LPT tokens to an unauthorized address.

**LivepeerGovernor**

The LivepeerGovernor contract is the main contract that implements the governance system for the Livepeer protocol. The LivepeerGovernor contract provides a number of features, such as:

* The ability to propose and vote on changes to the protocol
* The ability to execute proposals that have been approved by a majority of voters

The upgrade to the LivepeerGovernor contract has been carefully designed to be secure and robust. However, there are still some potential risks associated with the upgrade, such as:

* The LivepeerGovernor contract could be exploited to execute malicious proposals.
* The LivepeerGovernor contract could be exploited to censor votes from certain users.

**Conclusion**

The Livepeer Onchain Treasury Upgrade is a significant upgrade to the Livepeer protocol. The upgrade introduces a number of new features, such as an on-chain treasury, checkpointing of the BondingManager state, and support for delegators to override their delegates' votes.

The upgrade has been carefully designed to be secure and robust. However, there are still some potential risks associated with the upgrade, such as the treasury being drained or the voting power of users being manipulated.

### Time spent:
29 hours