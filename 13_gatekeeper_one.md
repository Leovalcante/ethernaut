# 13. Gatekeeper One
<sup>Difficulty 5/10</sup>

Make it past the gatekeeper and register as an entrant to pass this level.

Things that might help:

- Remember what you've learned from the Telephone and Token levels.
- You can learn more about the special function `gasleft()`, in Solidity's documentation (see [here](https://docs.soliditylang.org/en/v0.8.3/units-and-global-variables.html) and [here](https://docs.soliditylang.org/en/v0.8.3/control-structures.html#external-function-calls)).

## Solution

To register as an entrant we need to pass three modifiers:

```solidity
modifier gateOne() {
	require(msg.sender != tx.origin);
	_;
}

modifier gateTwo() {
	require(gasleft().mod(8191) == 0);
	_;
}

modifier gateThree(bytes8 _gateKey) {
  require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
  require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
  require(uint32(uint64(_gateKey)) == uint16(tx.origin), "GatekeeperOne: invalid gateThree part three");
	_;
}

function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
	entrant = tx.origin;
	return true;
}
```

Modifiers are something like a Python decorator: they wrap a function with some code.
In this case the three gate challenge us with the following five statements:

```solidity
require(msg.sender != tx.origin);                                                                           // 1
require(gasleft().mod(8191) == 0);                                                                          // 2
require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one"); // 3
require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");         // 4
require(uint32(uint64(_gateKey)) == uint16(tx.origin), "GatekeeperOne: invalid gateThree part three");      // 5
```

Let's find a solution for these challenges!

### Gate One

Gate one challenge `require(msg.sender != tx.origin);` is trivial. As we saw in previous challenge ([#04 Telephone](./04_telephone.md)) `msg.sender` get the address of whom makes the call, while `tx.origin` store the address of whom initiate the transaction. Also, `tx.origin` can never be a contract.

> In a simple call chain `A -> B -> C -> D`, inside `D`, `msg.sender`, will be `C`, and `tx.origin` will be `A`.

[1] The solution to the first gate is to call the `enter` function from another contract.

```solidity
contract GateBreaker {
    bytes8 public key;

    GatekeeperOne gko = GatekeeperOne(<addr>)

    function exploit() public {
        gko.enter(key);
    }
}
```

### Gate Two

Gate two challenge `require(gasleft().mod(8191) == 0);` requires us to manage our gas and to set a gas limit that is a multiple of 8191.

We can manually set this parameter in Remix IDE but we need to calculate how much gas we'll spend to verify the check.

Copy the `Gatekeeper One` contract into Remix IDE, then deploy 

TODO

### Gate Three

Gate three challenge us with three different problems:

1. `uint32(uint64(_gateKey)) == uint16(uint64(_gateKey))`
2. `uint32(uint64(_gateKey)) != uint64(_gateKey)`
3. `uint32(uint64(_gateKey)) == uint16(tx.origin)`

We start from point 3. as it's the only one with a variable we know: `tx.origin`.
In my case `uint16(tx.origin) == 17587`.

Bruteforce here TO EXPLAIN HOW, also check BETTER METHODS


## OZ Corner
