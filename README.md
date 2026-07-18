# Ethernaut Writeups

Solutions and security writeups for the [Ethernaut](https://ethernaut.openzeppelin.com) smart contract wargame by OpenZeppelin.

Each writeup follows the same structure: the vulnerability, the exploit, and the fix. The goal is to understand the root cause and how to prevent it in production code.

## Format

For every level:

- **Vulnerability** what is wrong with the contract
- **Exploit** the steps used to break it
- **Fix** how a secure version prevents the attack

## Levels

| # | Level | Vulnerability class |
| --- | --- | --- |
| 01 | Fallback | Access control, receive and fallback misuse |
| 02 | Fallout | Broken constructor access control |
| 03 | Coin Flip | Predictable on-chain randomness |
| 04 | Telephone | tx.origin vs msg.sender |
| 05 | Token | Integer underflow |

More levels added as they are solved.

## Tooling

- Solidity
- Foundry for local testing and exploit scripts

## Why this repo

Reviewing broken contracts and writing the fix is the core skill behind smart contract auditing and security bounties. This repo documents that practice, one level at a time.
