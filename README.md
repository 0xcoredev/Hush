# Hush

> A privacy-first anonymous confession platform on Midnight.
> Built for the Midnight Buildathon — Wave 2 submission.

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](./LICENSE)
[![Compact](https://img.shields.io/badge/Compact-0.27-purple.svg)](https://docs.midnight.network/compact)
[![Midnight](https://img.shields.io/badge/Midnight-nightntwrk-9b59b6.svg)](https://midnight.network)

Hush lets people post anonymous confessions, react to others, and have those
confessions moderated — all on Midnight, where zero-knowledge proofs make it
possible to verify identity, ownership, and authorization **without ever
revealing who anyone is or what they confessed**.

This repository contains the Wave 2 Compact contract and a simulation/smoke
test suite. The frontend and TypeScript SDK integration are scheduled for
Wave 3 (see [Roadmap](#roadmap)).

---

## Why Midnight

Anonymous confession platforms have historically failed in one of two ways:

1. **Pseudonymous-but-tracked** — the platform stores user identifiers that
   can be subpoenaed, leaked, or correlated across sessions.
2. **Fully anonymous-but-unmoderatable** — no way to remove abuse, spam, or
   harm without a central operator that re-identifies users.

Midnight's dual-ledger model resolves this tension. Public ledger state
(commitments, counters, flagged sets) is verifiable by anyone, while
zero-knowledge circuits prove authorization locally — the user's secret never
leaves their device. This lets Hush enforce rules ("only the original poster
can take down", "only a moderator can flag", "one reaction per user per
confession") without collecting any personally identifying data.

### Where Midnight's capabilities shape the design

| Midnight capability | How Hush uses it |
|---|---|
| **Compact smart contracts** | Single contract `contracts/hush.compact` implements the full confession lifecycle. |
| **Private state via witnesses** | `localSecretKey()` witness returns the user's secret off-chain; it is never disclosed in proofs or on the ledger. |
| **DApp-scoped public keys** | `persistentHash` over `"hush:pk:"` + secret derives a key bound to Hush only — the same user cannot be correlated across other Midnight DApps. |
| **Persistent commitments** | `persistentCommit<Bytes<32>>(contentHash, posterKey)` stores proof a confession exists and is owned by someone, without storing content or identity. |
| **Dual-ledger model** | Public ledger carries only commitments, counters, and sets. Original confession content lives off-chain with the poster. |
| **Explicit disclosure (`disclose()`)** | Every crossing of the privacy boundary is explicit and auditable in the source. |
| **Sealed ledger fields** | The moderator key commitment is `sealed`, so it can be read but not reassigned after construction. |
| **ZK proofs of role** | Moderators flag confessions by proving they hold the moderator DApp key — accountability without identity exposure. |

---

## Contract overview

File: [`contracts/hush.compact`](./contracts/hush.compact)

### Public ledger state

| Field | Type | Purpose |
|---|---|---|
| `moderator` | `Bytes<32>` (sealed) | DApp-scoped public key of the deploying moderator. |
| `confessionCommits` | `Set<Bytes<32>>` | Live confession commitments (active + flagged). |
| `takenDownCommits` | `Set<Bytes<32>>` | Commitments whose poster removed them. |
| `flaggedCommits` | `Set<Bytes<32>>` | Confessions a moderator has flagged. |
| `reactionCommits` | `Set<Bytes<32>>` | (reactor_key, confession_commit) commitments — enforces one reaction per user per confession. |
| `totalConfessions` | `Counter` | Public traction metric. |
| `totalReactions` | `Counter` | Public traction metric. |
| `totalFlags` | `Counter` | Public moderation metric. |

### Circuits

| Circuit | Visibility | What it does |
|---|---|---|
| `confess(contentHash, category)` | exported | Posts a confession; returns the on-chain commitment. Content stays off-chain. |
| `react(confessionCommit)` | exported | Anonymous reaction; double-react prevented via commitment set membership. |
| `takeDown(contentHash)` | exported | Poster proves ownership via regenerated commitment; moves it to `takenDownCommits`. |
| `flag(confessionCommit)` | exported | Moderator-only; proves moderator DApp key without revealing which moderator. |
| `unflag(confessionCommit)` | exported | Moderator-only; clears a flag. |
| `confessionCount()` | exported pure | Returns total confessions. |
| `reactionCount()` | exported pure | Returns total reactions. |
| `flagCount()` | exported pure | Returns total flags. |
| `getDappPublicKey(_secret)` | internal | Derives DApp-scoped key via `persistentHash`. |

### Privacy boundary

The only values ever crossing `disclose()` are:
- the moderator's DApp public key (once, in the constructor)
- confession commitments (already hash-derived)
- counter values (already public)

Everything else — the user's secret, the original confession text, the
content hash pre-image — remains in private state.

---

## Repository layout

```
hush/
├── contracts/
│   └── hush.compact        # The Compact smart contract
├── tests/
│   └── hush.simulation.js  # Source smoke test + managed API simulation
├── docs/
│   └── PITCH.md            # Slide deck outline (Wave 2 submission)
├── package.json
├── LICENSE                 # Apache 2.0
├── .gitignore
└── README.md
```

---

## Setup

### Prerequisites

- Node.js 20+ and npm >= 9
- The Compact compiler (`compact`) — see the
  [Midnight toolchain install guide](https://docs.midnight.network/examples/getting-started/installation)
- Docker Desktop (only required for running the proof server against Preprod)

### Install

```bash
npm install
```

### Compile the contract

```bash
npm run compact
```

This runs:

```bash
compact compile contracts/hush.compact src/managed/hush
```

On success you will see output similar to:

```
Compiling N circuits:
  circuit "confess" (k=..., rows=...)
  circuit "react"  (k=..., rows=...)
  ...
```

The managed TypeScript API is emitted to `src/managed/hush/`.

### Run the tests

```bash
npm test
```

The test suite has two layers:

1. **Source smoke test** — always runs. Validates that `hush.compact` exists,
   imports `CompactStandardLibrary`, pins a language version, and declares
   every required circuit, ledger field, and enum. This documents and
   protects the Wave 2 surface area.
2. **Managed API simulation** — runs only after `npm run compact` has
   produced `src/managed/hush`. Exercises the lifecycle
   `confess -> react -> flag -> takeDown` against the compiled contract.

### Lint / type-check the contract

```bash
npm run lint
```

This runs `compact check contracts/hush.compact` to surface syntax and type
errors without emitting artifacts.

---

## How judges can evaluate

1. **Compile gate** — `npm run compact` must succeed. This satisfies the
   Buildathon's technical gate ("at least one Compact contract that compiles
   successfully").
2. **Tests** — `npm test` must pass. The source smoke test always passes if
   the contract file is intact; the managed API simulation passes once the
   compiler has emitted artifacts.
3. **Privacy design review** — read
   [`contracts/hush.compact`](./contracts/hush.compact). Every `disclose()`
   call is intentional and auditable. The contract never stores confession
   content or user identity on the public ledger.
4. **Slide deck** — see [`docs/PITCH.md`](./docs/PITCH.md).
5. **Demo video** — record `npm run compact && npm test` plus a walkthrough
   of the contract's privacy boundary. The video pitch should explain why
   each circuit uses `persistentCommit` / `persistentHash` rather than
   storing raw data.

---

## Roadmap

### Wave 2 (this submission)

- ✅ Compact contract: `confess`, `react`, `takeDown`, `flag`, `unflag`,
  `confessionCount`, `reactionCount`, `flagCount`
- ✅ Private state via `localSecretKey` witness
- ✅ DApp-scoped public keys via `persistentHash`
- ✅ Content commitments via `persistentCommit`
- ✅ ZK-moderated flagging (moderator proves role without revealing identity)
- ✅ Source smoke test + managed API simulation scaffold
- ✅ Apache 2.0 license, `midnightntwrk` GitHub topic

### Wave 3 (next)

- TypeScript SDK integration using `@midnight-ntwrk/sdk`
- End-to-end frontend (Next.js) with:
  - confession composer (computes content hash client-side)
  - anonymous reaction buttons
  - moderator dashboard (proves moderator key locally, no identity reveal)
- Local proof server integration for Preprod deployment
- Optional shielded tipping via `nativeToken()` and `sendUnshielded` /
  `receiveUnshielded`
- Optional encrypted confession content storage using a
  DApp-scoped symmetric key derived from the user's secret, so confessions
  can be selectively revealed to specific parties (e.g. a therapist, a
  compliance officer) without going on-chain
- Categorized commitment index for filtered browsing

### Beyond Wave 3

- Cross-DApp reputation: prove "I have had N confessions accepted by
  moderators across Hush and partner DApps" without revealing which
  confessions or which DApps
- Selective disclosure credentials: "prove I am a verified health
  professional" before posting in the HEALTH category, without revealing
  identity
- Interoperability with Cardano identity attestations

---

## Threat model (summary)

| Threat | Mitigation |
|---|---|
| Confession content leaks | Content never stored on-chain; only `persistentCommit` over its hash. |
| Poster identity revealed | Identity is a DApp-scoped `persistentHash` of a local secret; untrackable across DApps. |
| Double-reaction / sybil reactions | `reactionCommits` set enforces one reaction per (user, confession) pair via commitment membership. |
| Unauthorized take-down | `takeDown` regenerates the commitment from the caller's secret; non-owners fail the membership check. |
| Unauthorized moderation | `flag` / `unflag` require `moderator == callerKey`; moderator key is `sealed` after construction. |
| Moderator identity exposed | Moderator proves ownership of the DApp key in-ZK; no on-chain identifier beyond the key commitment. |
| Replay of stale posts | `persistentCommit` is content-bound; identical content by the same user yields the same commitment and is rejected. |

A fuller threat model will accompany the Wave 3 frontend submission.

---

## License

All Midnight-related code in this repository is licensed under
[Apache License 2.0](./LICENSE), as required by the Midnight Buildathon
Official Rules.

## Acknowledgements

- Built on [Midnight](https://midnight.network) using
  [Compact](https://docs.midnight.network/compact).
- Contract patterns informed by the Midnight docs examples
  (private guest list, bulletin board).
