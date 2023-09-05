| *Issue*      | *Description*                                                                                                                                                                                 |
|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **[ L-01 ]** | [setNumActiveTranscoders](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L186-L190) size can only be increased                                 |
| **[ L-02 ]** | [tryToJoinActiveSet](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1407-L1409) does not return an event when it cannot plug a new transcoder |

### [L-01] [setNumActiveTranscoders](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L186-L190) size can only be increased
Due to how [SortedDoublyLL](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/libraries/SortedDoublyLL.sol) active transcoders max size can only increase. For now this will not cause any issues, however in the future the team might want to decrease the size, and this will not be possible.

Example:
1. Not many people are transcoders, so the devs set the size to 500
2. The system expands and becomes widely used with many participants
3. Devs want to limit the active transcoders to 300, but that will not be possible 

My suggesting is to make a function to modify the number of active transcoders while the system is not fully out of production.

### [L-02] [tryToJoinActiveSet](https://github.com/code-423n4/2023-08-livepeer/blob/main/contracts/bonding/BondingManager.sol#L1407-L1409) does not return an event when it cannot plug a new transcoder 
When a delegate bonds to himself he becomes a transcoder. When that happens the system tries to push him to the active transcoders list, however if he is not able to join that list, the function just returns an empty `return`. 
```jsx
            if (_totalStake <= lastStake) {
                return;//@audit returns without an event, user does not know if he is or not a transcoder
            }
```
It would be better for it to return an event signaling that he does not have enought delegated amount to become a transcoder.