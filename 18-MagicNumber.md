# Level 18: Magic Number

## Goal

Provide a `Solver` contract whose `whatIsTheMeaningOfLife()` returns 42, but the runtime bytecode must be **10 bytes or fewer**. That is too small for a normal Solidity contract, so we write raw EVM bytecode.

## The contract

```solidity
contract MagicNum {
    address public solver;
    function setSolver(address _solver) public {
        solver = _solver;
    }
}
```

The verifier calls the solver and expects the 32-byte value 42 back.

## The runtime bytecode (10 bytes)

Return 42 regardless of input:

```
602a    PUSH1 0x2a   // 42
6000    PUSH1 0x00
52      MSTORE       // memory[0..32] = 42
6020    PUSH1 0x20   // length 32
6000    PUSH1 0x00   // offset 0
f3      RETURN
```

Runtime = `602a60005260206000f3` (10 bytes).

## The creation (init) bytecode

The init code copies the runtime into memory and returns it:

```
600a    PUSH1 0x0a   // runtime length 10
600c    PUSH1 0x0c   // runtime starts at offset 12 in this init code
6000    PUSH1 0x00
39      CODECOPY
600a    PUSH1 0x0a
6000    PUSH1 0x00
f3      RETURN
```

Full creation bytecode = `600a600c600039600a6000f3` + `602a60005260206000f3`
= `0x600a600c600039600a6000f3602a60005260206000f3`

## Exploit

```js
// Ethernaut console
const bytecode = "0x600a600c600039600a6000f3602a60005260206000f3";
const txn = await web3.eth.sendTransaction({ from: player, data: bytecode });
await contract.setSolver(txn.contractAddress);
```

## Fix

This level is educational rather than a "bug." The lesson is understanding the EVM at the bytecode level: opcodes, memory, and how contract creation returns runtime code.

## Takeaway

Solidity is just a front end for EVM bytecode. Being able to read and hand-write opcodes is essential for advanced auditing (gas golf, proxies, obfuscated exploits).
