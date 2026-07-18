# Level 01: Fallback

## Goal

Claim ownership of the contract and reduce its balance to zero.

## The contract

Ownership is meant to require contributing more than the owner (1000 ether). But the `receive` function has a much weaker check:

```solidity
function contribute() public payable {
  require(msg.value < 0.001 ether);
  contributions[msg.sender] += msg.value;
  if (contributions[msg.sender] > contributions[owner]) {
    owner = msg.sender;
  }
}

receive() external payable {
  require(msg.value > 0 && contributions[msg.sender] > 0);
  owner = msg.sender;
}
```

## Vulnerability

The `receive` function grants ownership to anyone who has made any contribution and sends any amount of ether. It never checks the intended 1000 ether threshold. So a single tiny contribution plus a bare ether transfer is enough to take over.

## Exploit

1. Call `contribute()` with a value below 0.001 ether. Now `contributions[attacker] > 0`.
2. 2. Send a plain ether transfer to the contract with any non zero value. This triggers `receive`, which sets `owner = attacker`.
   3. 3. Call `withdraw()` as the new owner to drain the balance to zero.
     
      4. ## Fix
     
      5. Never assign privileged roles inside `receive` or `fallback`. Ownership changes should go through an explicit, access controlled function:
     
      6. ```solidity
         receive() external payable {}

         function claimOwnership() external {
           require(contributions[msg.sender] > contributions[owner], "not enough");
           owner = msg.sender;
         }
         ```

         ## Takeaway

         `receive` and `fallback` run on plain ether transfers and should stay minimal. Putting authorization logic there is a classic access control bug.
         
