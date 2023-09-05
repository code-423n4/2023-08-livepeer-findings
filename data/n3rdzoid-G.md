
# Cost-Efficient Error Handling in Solidity 0.8.4: Custom Errors vs. Revert Strings

In Solidity version 0.8.4, utilizing custom errors proves to be more cost-effective compared to using revert strings, resulting in reduced deployment and runtime expenses when the revert condition is triggered. Custom errors help conserve approximately 50 gas each time they are triggered since they eliminate the need to allocate and store the revert string. Furthermore, not defining these strings also contributes to savings in deployment costs.


File name = ![](https://github.com/code-423n4.png?size=32)
[contracts/ManagerProxy.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/ManagerProxy.sol#L52)
require(target != address(0), "target contract must be registered");
require(delegationAmount > 0, "delegation amount must be greater than 0");

|require(isValidUnbondingLock(msg.sender, _unbondingLockId), "invalid unbonding lock ID");|

File Name = ![](https://github.com/code-423n4.png?size=32)
[contracts/bonding/BondingManager.sol](https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L253)

|   |
|---|
|require(_recipient != address(0), "invalid recipient");|


|   |
|---|
|require(fees >= _amount, "insufficient fees to withdraw");|

|   |
|---|
|require(isRegisteredTranscoder(_transcoder), "transcoder must be registered");|


|   |
|---|
|require(!roundsManager().currentRoundLocked(), "can't update transcoder params, current round is locked");|

|   |
|---|
|require(MathUtils.validPerc(_rewardCut), "invalid rewardCut percentage");|

|   |
|---|
|require(MathUtils.validPerc(_feeShare), "invalid feeShare percentage");|

|   |
|---|
|require(isRegisteredTranscoder(msg.sender), "transcoder must be registered");|
||   |

|   |
|---|
|require(_to != _owner, "INVALID_DELEGATE");|

|   |
|---|
|require(currentDelegate == _to, "INVALID_DELEGATE_CHANGE");|

|   |
|---|
|require(!isRegisteredTranscoder(_owner), "registered transcoders can't delegate towards other addresses");|

|   |
|---|
|require(delegationAmount > 0, "delegation amount must be greater than 0");|

|   |
|---|
|require(oldDelDelegate != _delegator, "INVALID_DELEGATOR");|


|   |
|---|
|require(delegatorStatus(msg.sender) == DelegatorStatus.Bonded, "caller must be bonded");|
||   |

require(_amount > 0, "unbond amount must be greater than 0");
require(_amount <= del.bondedAmount, "amount is greater than bonded amount");
require(delegatorStatus(msg.sender) != DelegatorStatus.Unbonded, "caller must be bonded");


