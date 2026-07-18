# Level 06: Delegation

## Goal

Claim ownership of the Delegation contract.

## The contracts

```solidity
contract Delegate {
    address public owner;
    function pwn() public { owner = msg.sender; }
}

contract Delegation {
    address public owner;
    Delegate delegate;

    fallback() external {
        (bool result,) = address(delegate).delegatecall(msg.data);
        if (result) { this; }
    }
}
```

## Vulnerability

`Delegation`'s fallback forwards **any** calldata to `Delegate` via `delegatecall`. With `delegatecall`, the delegate's code runs in the **caller's storage context**. So `Delegate.pwn()`, executed through delegatecall, writes to **Delegation's** `owner` slot, not Delegate's.

Since the fallback runs for arbitrary `msg.data`, an attacker just needs to send the function selector for `pwn()`.

## Exploit

1. Compute the selector for `pwn()`: first 4 bytes of `keccak256("pwn()")` = `0xdd365b8b`.
2. Send a transaction to the Delegation contract with that as the data:

```js
// Ethernaut console
await contract.sendTransaction({ data: "0xdd365b8b" });
```

3. The fallback delegatecalls `pwn()`, which sets `Delegation.owner = you`.

## Fix

- Do not `delegatecall` arbitrary, caller-supplied calldata.
- If delegation is required, whitelist exactly which functions may be delegated, and never let untrusted code write privileged storage slots.
- Be aware that `delegatecall` shares storage layout, so slot collisions are dangerous.

## Takeaway

`delegatecall` executes foreign code against your own storage. Forwarding arbitrary calldata into it hands an attacker a write primitive over your contract's state.
