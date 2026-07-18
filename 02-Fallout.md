# Level 02: Fallout

## Goal

Claim ownership of the contract.

## The contract

The contract is named `Fallout`. Ownership is set in what looks like the constructor:

```solidity
/* constructor */
function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
}
```

## Vulnerability

This was written for an old Solidity version where the constructor had to share the exact name of the contract. The contract is `Fallout`, but the function is `Fal1out` (a digit `1` instead of the letter `l`).

Because the names do not match, this is not a constructor. It is an ordinary public function that anyone can call, and it sets `owner = msg.sender`.

## Exploit

1. Call `Fal1out()` directly. Since it is a normal public function with no access control, the caller becomes `owner`.

That single call takes ownership.

## Fix

Modern Solidity uses the `constructor` keyword, which removes this entire class of bug:

```solidity
constructor() payable {
    owner = msg.sender;
}
```

Never rely on a function name to make something a constructor, and always confirm that privileged setup functions are actually constructors and not callable afterwards.

## Takeaway

A single character (`1` vs `l`) turned a locked constructor into an open door. Careful reading of privileged function names is a real audit skill.
