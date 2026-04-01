# 🔗 MARS Chain Node

A decentralized verification node for the MARS virtual token economy.

This repo is one node in a network of independent verifiers. Each node:
1. **Receives** cartridge submissions (attestations of colony survival runs)
2. **Replays** them against the [public frame ledger](https://github.com/kody-w/mars-barn-opus/tree/main/data/frames)
3. **Verifies** the hash chain integrity (every block links to the previous)
4. **Records** verified runs in `chain/verified/`
5. **Publishes** consensus state via GitHub Pages

## Architecture

```
┌─────────────────────────────────┐
│  mars-barn-opus (main repo)     │
│  data/frames/*.json (the truth) │
│  Public frame ledger            │
└──────────────┬──────────────────┘
               │ frames
    ┌──────────┼──────────┐
    ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐
│ Node 1 │ │ Node 2 │ │ Node N │  ← independent repos
│ verify │ │ verify │ │ verify │     anyone can fork & run
│ record │ │ record │ │ record │
└────────┘ └────────┘ └────────┘
    │          │          │
    ▼          ▼          ▼
  chain/     chain/     chain/
  verified/  verified/  verified/
    │          │          │
    └──────────┼──────────┘
               ▼
         CONSENSUS:
    If 2/3 nodes agree on
    a chain head → CONFIRMED
```

## How It Works

### Verification Protocol

1. Player exports a cartridge from the sim (`.cartridge.json`)
2. Player submits it to any node (drops file in `submissions/` via PR or Action)
3. Node's GitHub Action:
   - Fetches the public frames the cartridge claims to have consumed
   - Verifies frame hashes match the public ledger
   - Verifies the hash chain (each block's `prevHash` links correctly)
   - Checks token rewards match the token economics LisPy program
   - If valid: moves to `chain/verified/` with a verification receipt
   - If invalid: moves to `chain/rejected/` with the reason

### Chain State

```
chain/
├── genesis.json          # Genesis block (network parameters)
├── state.json            # Current chain head, circulating supply
├── verified/             # Verified cartridge attestations
│   ├── MBC-xxx.json      # One file per verified run
│   └── ...
├── rejected/             # Failed verifications (with reasons)
└── peers.json            # Known peer nodes for cross-verification
```

### Token Rules (enforced by verification)

- **Total supply:** 21,000,000 MARS
- **Halving:** Reward halves every 500 sols
- **Scarcity curve:** Early sols earn more (< 1% supply minted = 3× reward)
- **Per-agent:** Tokens belong to virtual agents, not users
- **Deterministic:** Same frames + same decisions = same rewards (replay-verifiable)

## Running Your Own Node

1. **Fork this repo**
2. Enable GitHub Actions
3. Enable GitHub Pages (from `docs/` folder)
4. Add your node to the peer registry by PRing `peers.json` in any existing node
5. Your node now independently verifies submissions

No server. No infrastructure. Just a GitHub repo with Actions.

## Verification API

Each node serves verification results via GitHub Pages:

```
https://<user>.github.io/mars-chain-node/api/state.json     # chain state
https://<user>.github.io/mars-chain-node/api/verified.json   # verified runs
https://<user>.github.io/mars-chain-node/api/peers.json      # known peers
```

## Cross-Verification

Nodes check each other's work:
- Each node periodically fetches peer `state.json`
- If chain heads diverge, nodes compare block-by-block
- Consensus = 2/3 of nodes agree on the chain head
- Disagreement is logged and surfaced to the SimHub

## Related

- [Mars Barn Opus](https://github.com/kody-w/mars-barn-opus) — the sim
- [SimHub](https://kody-w.github.io/mars-barn-opus/simhub.html) — leaderboard
- [Pattern Library](https://kody-w.github.io/mars-barn-opus/patterns.html) — engineering patterns
