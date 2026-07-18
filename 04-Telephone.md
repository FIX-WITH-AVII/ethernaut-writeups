# Level 04: Telephone

## Goal

Claim ownership of the contract.

## The contract

```solidity
function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
        owner = _owner;
    }
}
```

## Vulnerability

The contract authorizes the ownership change based on `tx.origin != msg.sender`.

- `tx.origin` is the original externally owned account (EOA) that started the whole transaction.
- `msg.sender` is the immediate caller (could be a contract).

If you call `changeOwner` directly from your wallet, `tx.origin` and `msg.sender` are both you, so they are equal and the condition fails. But if you call it **through an intermediate contract**, `tx.origin` is still your wallet while `msg.sender` is the intermediate contract, so they differ and the check passes.

## Exploit

1. Deploy an attacker contract with a function that calls the target:

```solidity
contract Attack {
    function pwn(Telephone target) external {
        target.changeOwner(msg.sender);
    }
}
```

2. Call `pwn()` from your wallet. Now inside `changeOwner`, `tx.origin` is your wallet and `msg.sender` is the Attack contract, so `tx.origin != msg.sender` is true and you become owner.

## Fix

Never use `tx.origin` for authorization. Use `msg.sender`:

```solidity
function changeOwner(address _owner) public {
    require(msg.sender == owner, "not owner");
    owner = _owner;
}
```

`tx.origin` auth is also dangerous because it enables phishing: a malicious contract can trick a user into a transaction that passes a `tx.origin` check.

## Takeaway

`tx.origin` vs `msg.sender` is a fundamental distinction. Authorization should always be based on the immediate caller (`msg.sender`), never the transaction origin.
