# MASHED

A boss-fight game built for Monad Blitz Hyderabad. A slugcat descends into
ruined depths to face The Warden — a knight who calls on echoes of himself
when wounded — and every victory is recorded permanently on the Monad
blockchain: attempts, clear time, and heals used, verifiable by anyone.

## The problem this solves

Claims of skill in games, speedruns, and competitive challenges are usually
unverifiable — a screenshot can be faked, a leaderboard can be gamed. MASHED
records the exact conditions of every win (how many tries it took, how fast,
how many heals were burned) as a permanent, tamper-proof entry on-chain. It's
a small, concrete demonstration of a pattern that generalizes to any
skill-based claim that currently relies on trust: verification without a
trusted intermediary.

## How to play

Open `mashed.html` in a browser. No install required.

| Action | Key |
|---|---|
| Move | ← → |
| Jump | Space |
| Dash / dodge | S |
| Charge attack (hold, release to swing) | Hold Left-Click |
| Block / parry | Right-Click or W |
| Use potion (heals 50%, 3 max) | Z |

## Core mechanics

- **Charge attacks** — hold to charge, release to swing; a fully charged hit
  deals more damage.
- **Block & parry** — raising your guard is a real reverse-swing animation,
  not an instant toggle; block reach grows as the motion completes, so a
  late block genuinely doesn't cover you yet. Parrying inside a short window
  staggers the boss and opens a big damage window.
- **Stamina** — attacks, jumps, and dashes all draw from a shared stamina
  bar (6 swings / 8 jumps / 3 dashes from a full bar), which regenerates
  after a short pause. Forces real resource management under pressure.
- **The Warden (boss)** — phase-based AI with telegraphed swipes, slam
  attacks, and ranged spit; chains combo swipes more aggressively in phase 2.
- **Echoes (ghosts)** — permanent, aggressive clones spawned at HP
  thresholds (75/50/25%). They chase, attack on their own, and only leave
  the fight when killed. A single swing can hit the boss and any overlapping
  ghosts together.
- **Speedrun timer** — tracks time from the first move to victory, shown
  live and recorded in the final result.

## On-chain victory log

On winning, the game can mint a permanent record via a smart contract,
`MashedVictoryLog.sol`, deployed to Monad Testnet:

```solidity
function recordVictory(uint256 attempts, uint256 timeMs, uint256 healsUsed) external
function getVictoryCount(address player) external view returns (uint256)
function getVictory(address player, uint256 index) external view returns (Victory memory)
```

Each victory is stored as a `Victory { attempts, timeMs, healsUsed, timestamp }`
struct under the caller's address, plus an emitted `VictoryRecorded` event.
Anyone can independently verify a claimed result by querying the contract.

**Status:** the contract is written, compiles cleanly, and the game's
`mintOnChain()` function is fully wired to call it via ethers.js — it falls
back to a simulated mint automatically if no contract address is configured,
so the game is always demoable end-to-end regardless of deployment status.
Live deployment to Monad Testnet was blocked during the event by wallet/IDE
tooling issues (browser extension state getting out of sync across tabs),
not by anything in the game or contract logic itself.

## Tech stack

- Vanilla JS + HTML5 Canvas — no build step, no dependencies for the game
  itself.
- Solidity ^0.8.19 for the on-chain victory log.
- ethers.js (via CDN) for the wallet/contract connection.

## Deploying the contract yourself

1. Open [Remix](https://remix.ethereum.org), create `MashedVictoryLog.sol`,
   paste in the contract source.
2. Compile with EVM version **paris** (Monad Testnet compatibility).
3. Add Monad Testnet to MetaMask: RPC `https://testnet-rpc.monad.xyz`,
   Chain ID `10143`, currency `MON`, explorer
   `https://testnet.monadexplorer.com`.
4. Get testnet MON from `https://faucet.monad.xyz`.
5. In Remix's Deploy tab, set environment to **Injected Provider – MetaMask**,
   deploy, and confirm in MetaMask.
6. Copy the deployed address and ABI into `CONTRACT_ADDRESS` and
   `CONTRACT_ABI` in `mashed.html`.

## Credits

Built solo for Monad Blitz Hyderabad.
