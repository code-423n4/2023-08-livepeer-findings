# L-01: `bondWithHint` function is unnecessary .
The function `bondWithHint` is called from `bond` function . After calling it calls `bondForWithHint` function with `msg.sender` as the owner .   `bondForWithHint` function can be directly called from the `bond` function with `msg.sender` as the owner . Unnecessary functions are recommended to avoid to reduce code complexity and size. 
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L640

# L-02: Events not emitting the updated value in `setUnbondingPeriod` function 
The event in  `setUnbondingPeriod` function  doesnot emit the updated value of new `unbondingPeriod` value . Which may cause problems in offchain integration of the protocol . 
```solidity
    function setUnbondingPeriod(uint64 _unbondingPeriod) external onlyControllerOwner {
        unbondingPeriod = _unbondingPeriod;

        emit ParameterUpdate("unbondingPeriod"); //@a  value not emitted ? 
    }

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L155

# L-03: Events not emitting the updated value in `setTreasuryBalanceCeiling` function 

The event in  `setTreasuryBalanceCeiling ` function  doesnot emit the updated value of new `treasuryBalanceCeiling` value . Which may cause problems while maintaining  offchain integration of the protocol . 
```solidity

    function setTreasuryBalanceCeiling(uint256 _ceiling) external onlyControllerOwner {
        treasuryBalanceCeiling = _ceiling;

        emit ParameterUpdate("treasuryBalanceCeiling"); //@a QA value  not emitted ? 
    }

```
# L-04: Events not emitting the updated value in `setNumActiveTranscoders` function 
The event in  `setNumActiveTranscoders` function  doesnot emit the updated value of new  maximum number of active transcoders value . Which may cause problems while maintaining  offchain integration of the protocol 
```solidity
   function setNumActiveTranscoders(uint256 _numActiveTranscoders) external onlyControllerOwner {
        transcoderPool.setMaxSize(_numActiveTranscoders);

        emit ParameterUpdate("numActiveTranscoders"); //<---updated value not emitted here 
    }

```
https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L186