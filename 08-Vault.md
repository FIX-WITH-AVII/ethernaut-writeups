# Level 08: Vault

## Goal

Unlock the vault (set `locked` to false).

## The contract

```solidity
contract Vault {
    bool public locked;
    bytes32 private password;

    constructor(bytes32 _password) {
        locked = true;
        password = _password;
    }

    function unlock(bytes32 _password) public {
        if (password == _password) {
            locked = false;
        }
    }
}
```

## Vulnerability

`password` is marked `private`, but `private` only controls **Solidity-level visibility**. It does **not** hide the value on-chain. Every storage slot of every contract is publicly readable directly from the blockchain.

Storage layout:
- slot 0: `locked`
- slot 1: `password`

Anyone can read slot 1 and recover the password.

## Exploit

```js
// Ethernaut console
const password = await web3.eth.getStorageAt(contract.address, 1);
await contract.unlock(password);
```

Reading slot 1 gives the exact bytes32 password, which we pass straight to `unlock`.

## Fix

- Never store secrets on-chain in plaintext. Anything on-chain is public, forever.
- If you must commit to a secret, store only a **hash** of it (commit-reveal), or keep the secret off-chain and verify a signature.

## Takeaway

`private` is not `secret`. It hides a variable from other contracts' code, not from anyone reading the chain. On-chain data has no confidentiality.
