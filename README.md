# Ethernaut Writeups

Solutions and security writeups for the [Ethernaut](https://ethernaut.openzeppelin.com) smart contract wargame by OpenZeppelin.

Each writeup follows the same structure: the vulnerability, the exploit, and the fix. The goal is to understand the root cause and how to prevent it in production code.

## Format

For every level:

1. Vulnerability, what is wrong with the contract
2. 2. Exploit, the steps used to break it
   3. 3. Fix, how a secure version prevents the attack
     
      4. ## Levels
     
      5. | # | Level | Vulnerability class |
      6. | --- | --- | --- |
      7. | 01 | Fallback | Access control, receive and fallback misuse |
      8. | 02 | Fallout | Broken constructor access control |
      9. | 03 | Coin Flip | Predictable on chain randomness |
      10. | 04 | Telephone | tx.origin vs msg.sender |
      11. | 05 | Token | Integer underflow |
     
      12. More levels added as they are solved.
     
      13. ## Tooling
     
      14. - Solidity
          - - Foundry for local testing and exploit scripts
           
            - ## Why this repo
           
            - Reviewing broken contracts and writing the fix is the core skill behind smart contract auditing and security bounties. This repo documents that practice, one level at a time.
            - 
