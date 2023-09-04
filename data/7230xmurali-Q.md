## Impact
maximum number of active transcoders must and should be != 0.

## Proof of Concept

function setNumActiveTranscoders(uint256 _numActiveTranscoders) external onlyControllerOwner {
        transcoderPool.setMaxSize(_numActiveTranscoders);

        emit ParameterUpdate("numActiveTranscoders");
    }

when the maximum number of active transcoders is Equal to 0. then there is no transcoders . when no transcoders then it leads to the cause of unbehavior.so check that _numActiveTranscoders value is !=0.

## Tools Used
manual

## Recommended Mitigation Steps
check that _numActiveTranscoders = 0 then revert 

function setNumActiveTranscoders(uint256 _numActiveTranscoders) external onlyControllerOwner {
       ++ require(_numActiveTranscoders != 0 ,"_numActiveTranscoders is Equal to 0");
        transcoderPool.setMaxSize(_numActiveTranscoders);

        emit ParameterUpdate("numActiveTranscoders");
    }
And Also i Think that it is better to give a max value to the like 2**256 - 1 because so many no. of transcoders are joining day by day right so the value is increasing.