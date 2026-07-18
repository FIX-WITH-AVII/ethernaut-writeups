# Level 03: Coin Flip

## Goal

Guess the coin flip correctly 10 times in a row.

## The contract

```solidity
uint256 blockValue = uint256(blockhash(block.number - 1));
uint256 coinFlip = blockValue / FACTOR;
bool side = coinFlip == 1 ? true : false;

if (side == _guess) {
    consecutiveWins++;
} else {
    consecutiveWins = 0;
}
```

## Vulnerability

The "random" coin flip is derived entirely from **public on-chain values**: `blockhash(block.number - 1)` and the constant `FACTOR`. Nothing here is secret or unpredictable. Anyone can read the same block hash and run the same math.

The auditor question that finds this: *can an attacker see or control the values this contract depends on?* Here the answer is yes, completely.

## Exploit

1. Deploy an attacker contract that computes the flip the exact same way, in the same transaction/block:

```solidity
uint256 blockValue = uint256(blockhash(block.number - 1));
bool guess = (blockValue / FACTOR) == 1;
coinFlip.flip(guess);
```

2. Because it runs in the same block, `blockhash(block.number - 1)` is identical to what the target sees, so the guess is always right.
3. Call it 10 times (across 10 blocks) to reach 10 consecutive wins.

## Fix

Never use on-chain data (`blockhash`, `block.timestamp`, `block.difficulty`) as a source of randomness. Use a verifiable external source such as **Chainlink VRF**, which returns randomness that cannot be predicted or reproduced by an attacker.

## Takeaway

On-chain "randomness" is not random. Every value a contract can read, an attacker can read too. Predictable randomness is a classic and costly bug in games, lotteries, and NFT mints.
