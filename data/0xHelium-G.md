## Gas1
The ```bond()``` function inside ```BondingManager.sol``` calls ```bondWithHint()``` which in turn calls ```bondForWithHint()``` which transfer livepeertoken (LPT) from the caller to the minter address .
https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L537-L623
https://github.com/code-423n4/2023-08-livepeer/blob/a3d801fa4690119b6f96aeb5508e58d752bda5bc/contracts/bonding/BondingManager.sol#L616
The problem here is: there is no check to ensure the caller have enough LPT to bond as he wishes . For instance if: caller wants to bond 1000 LPT but his LPT balance is 500, the call to bond() will revert but at the very end sweeping all the gas from caller without a tangible result:
# How to save gas inside this function ? :
Dev should early check that caller have at least amount tokens in his LPT balance to bond, if not early revert the function to save gas . This can be done using the balanceOf function from Oz ERC20 .
eg: if (livepeerToken().balanceOf(msg.sender) < amount) revert ACustomError();
