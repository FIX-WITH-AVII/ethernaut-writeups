# Ethernaut Writeups

Solutions and security writeups for the [Ethernaut](https://ethernaut.openzeppelin.com) smart contract wargame by OpenZeppelin.

Each writeup follows the same structure: the vulnerability, the exploit, and the fix. The goal is to understand the root cause and how to prevent it in production code.

## Format

For every level:

- **Vulnerability** what is wrong with the contract
- **Exploit** the steps (and code) used to break it
- **Fix** how a secure version prevents the attack

## Levels

| # | Level | Vulnerability class |
| --- | --- | --- |
| 01 | [Fallback](01-Fallback.md) | Access control, receive/fallback misuse |
| 02 | [Fallout](02-Fallout.md) | Broken constructor access control |
| 03 | [Coin Flip](03-CoinFlip.md) | Predictable on-chain randomness |
| 04 | [Telephone](04-Telephone.md) | tx.origin vs msg.sender |
| 05 | [Token](05-Token.md) | Integer underflow |
| 06 | [Delegation](06-Delegation.md) | delegatecall to arbitrary calldata |
| 07 | [Force](07-Force.md) | Forced ether via selfdestruct |
| 08 | [Vault](08-Vault.md) | Private storage is not secret |
| 09 | [King](09-King.md) | DoS via failing push payment |
| 10 | [Re-entrancy](10-Reentrancy.md) | Reentrancy |
| 11 | [Elevator](11-Elevator.md) | Untrusted external call, non-view interface |
| 12 | [Privacy](12-Privacy.md) | Storage packing / private not secret |
| 13 | [Gatekeeper One](13-GatekeeperOne.md) | gasleft, tx.origin, bit masking |
| 14 | [Gatekeeper Two](14-GatekeeperTwo.md) | extcodesize bypass in constructor |
| 15 | [Naught Coin](15-NaughtCoin.md) | ERC20 transferFrom bypasses lock |
| 16 | [Preservation](16-Preservation.md) | delegatecall storage collision |
| 17 | [Recovery](17-Recovery.md) | Deterministic addresses, open selfdestruct |
| 18 | [Magic Number](18-MagicNumber.md) | Raw EVM bytecode |
| 19 | [Alien Codex](19-AlienCodex.md) | Array length underflow -> storage write |
| 20 | [Denial](20-Denial.md) | DoS via gas exhaustion |
| 21 | [Shop](21-Shop.md) | Untrusted view call returns changing value |

## Tooling

- Solidity
- Foundry for local testing and exploit scripts

## Why this repo

Reviewing broken contracts and writing the fix is the core skill behind smart contract auditing and security bounties. This repo documents that practice, one level at a time.
# Ethernaut Writeups

Solutions and security writeups for the [Ethernaut](https://ethernaut.openzeppelin.com) smart contract wargame by OpenZeppelin.

Each writeup follows the same structure: the vulnerability, the exploit, and the fix. The goal is to understand the root cause and how to prevent it in production code.

## Format

For every level:

- **Vulnerability** what is wrong with the contract
- **Exploit** the steps (and code) used to break it
- **Fix** how a secure version prevents the attack

## Levels

| # | Level | Vulnerability class |
| --- | --- | --- |
| 01 | [Fallback](01-Fallback.md) | Access control, receive/fallback misuse |
| 02 | [Fallout](02-Fallout.md) | Broken constructor access control |
| 03 | [Coin Flip](03-CoinFlip.md) | Predictable on-chain randomness |
| 04 | [Telephone](04-Telephone.md) | tx.origin vs msg.sender |
| 05 | [Token](05-Token.md) | Integer underflow |
| 06 | [Delegation](06-Delegation.md) | delegatecall to arbitrary calldata |
| 07 | [Force](07-Force.md) | Forced ether via selfdestruct |
| 08 | [Vault](08-Vault.md) | Private storage is not secret |
| 09 | [King](09-King.md) | DoS via failing push payment |
| 10 | [Re-entrancy](10-Reentrancy.md) | Reentrancy |
| 11 | [Elevator](11-Elevator.md) | Untrusted external call, non-view interface |
| 12 | [Privacy](12-Privacy.md) | Storage packing / private not secret |
| 13 | [Gatekeeper One](13-GatekeeperOne.md) | gasleft, tx.origin, bit masking |
| 14 | [Gatekeeper Two](14-GatekeeperTwo.md) | extcodesize bypass in constructor |
| 15 | [Naught Coin](15-NaughtCoin.md) | ERC20 transferFrom bypasses lock |
| 16 | [Preservation](16-Preservation.md) | delegatecall storage collision |
| 17 | [Recovery](17-Recovery.md) | Deterministic addresses, open selfdestruct |
| 18 | [Magic Number](18-MagicNumber.md) | Raw EVM bytecode |
| 19 | [Alien Codex](19-AlienCodex.md) | Array length underflow -> storage write |
| 20 | [Denial](20-Denial.md) | DoS via gas exhaustion |

## Tooling

- Solidity
- Foundry for local testing and exploit scripts

## Why this repo

Reviewing broken contracts and writing the fix is the core skill behind smart contract auditing and security bounties. This repo documents that practice, one level at a time.
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
