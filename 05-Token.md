# Level 05: Token

## Goal

You start with 20 tokens. Take ownership of a much larger balance.

## The contract

```solidity
mapping(address => uint256) balances;

function transfer(address _to, uint256 _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
}
```

This contract uses an old Solidity version (before 0.8) with no overflow/underflow checks.

## Vulnerability

`balances` are `uint256`, which is **unsigned**: it can never be negative. So the check `balances[msg.sender] - _value >= 0` is **always true**, even when `_value` is larger than your balance.

Subtracting more than you have doesn't go negative, it **underflows** and wraps around to a huge number near 2^256.

## Exploit

1. You hold 20 tokens.
2. Call `transfer(anyAddress, 21)`.
3. The require passes (unsigned math never dips below zero). `balances[you]` becomes `20 - 21`, which underflows to roughly `2^256 - 1`, an enormous balance.

## Fix

- Use **Solidity 0.8+**, where arithmetic reverts automatically on overflow and underflow.
- On older versions, use **OpenZeppelin SafeMath**.
- Also, the `>= 0` check on an unsigned type is meaningless and should be `balances[msg.sender] >= _value`.

```solidity
// Solidity 0.8+ checked math
function transfer(address _to, uint256 _value) public returns (bool) {
    require(balances[msg.sender] >= _value, "insufficient");
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
}
```

## Takeaway

Integer underflow/overflow was one of the most damaging bug classes before Solidity 0.8. Always use checked math, and be suspicious of any comparison against zero on an unsigned integer.
