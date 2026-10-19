# Hush — Wave 2 Submission Narrative

> Responses to the AKINDO submission prompts. Paste each section into the
> corresponding field on the AKINDO submission form.

## What it does

Hush is a privacy-first anonymous confession platform on Midnight. Users can post confessions, react to others, and have those confessions moderated — all without ever revealing who they are or what they confessed.

The Wave 2 submission ships a compiling Compact contract (`contracts/hush.compact`) with 8 circuits covering the full confession lifecycle:

- **`confess(contentHash, category)`** — posts a confession. Only a commitment over the content hash + the poster's DApp-scoped public key is stored on-chain; the content itself never touches the ledger.
- **`react(confessionCommit)`** — anonymous reaction. One reaction per (user, confession) pair is enforced via a commitment set, preventing double-reactions without revealing who reacted.
- **`takeDown(contentHash)`** — the original poster proves ownership by regenerating the same commitment from their local secret, without revealing identity. The commitment moves to `takenDownCommits`; no content trace remains.
- **`flag(confessionCommit)` / `unflag(...)`** — a moderator proves they hold the moderator DApp key in-ZK, flagging or unflagging a confession. No one observing the chain learns which moderator acted.
- **`confessionCount` / `reactionCount` / `flagCount`** — public counters for traction and moderation metrics.

## The problem it solves

Anonymous confession platforms have historically failed in one of two ways:

1. **Pseudonymous-but-tracked** — the platform stores user identifiers (wallets, emails, session cookies) that can be subpoenaed, leaked, or correlated across sessions. The "anonymity" is a lie. Users don't trust the platform.
2. **Fully anonymous-but-unmoderatable** — no way to remove abuse, spam, or harm without a central operator that re-identifies users. The platform becomes unusable.

Existing Web2 confession apps (and most Web3 attempts) sit on one end of this spectrum. They either collect identity to enforce rules, or they enforce no rules at all.

Hush resolves this tension using Midnight's dual-ledger model. Public ledger state (commitments, counters, flagged sets) is verifiable by anyone, while zero-knowledge circuits prove authorization locally — the user's secret never leaves their device. Hush enforces "only the original poster can take down", "only a moderator can flag", and "one reaction per user per confession" **without collecting any personally identifying data**.

## Challenges I ran into

- **Compact's privacy boundary is strict and explicit.** The compiler rejects any caller-supplied parameter passed to a ledger operation (`.member()`, `.insert()`, `.remove()`) unless it's wrapped in `disclose()`. This caught us on the `react`, `flag`, and `unflag` circuits — passing `confessionCommit` directly to `assert(set.member(...))` failed compilation with a "potential witness-value disclosure" error. The fix was to wrap each caller-supplied value in `disclose()` at the moment it crosses into ledger state, since commitments are already hash-derived and intentionally public.
- **`pure` circuits cannot read ledger state.** The first compile failed because the counter circuits (`confessionCount`, `reactionCount`, `flagCount`) were marked `pure` but read from `Counter` ledger fields. Removed the `pure` modifier — read-only does not mean pure in Compact's model.
- **Distinguishing which values need `disclose()` and which don't.** Values derived from `persistentCommit` and `persistentHash` are ledger-safe (the compiler treats them as already-hash-derived), but caller-supplied values are not. This distinction is the heart of Midnight's privacy model and took a full compile-debug cycle to internalize.
- **Pinning the language version.** Compact is evolving; `pragma language_version 0.23;` was required for compatibility with the installed compiler (0.5.3 toolchain).
- **Limited time.** Wave 2 had a hard deadline; we deliberately scoped the frontend out to Wave 3 to protect the technical gate (compiling contract) and the highest-weighted rubric criterion (Engineering, 40%).

## Technologies I used

- **Midnight** — the blockchain platform (dual-ledger, ZK-native)
- **Compact** — Midnight's smart-contract language (v0.23 language version, v0.5.3 compiler)
- **CompactStandardLibrary** — `persistentCommit`, `persistentHash`, `Set<Bytes<32>>`, `Counter`, `pad`, `disclose`
- **TypeScript / Node.js** — test runner and simulation harness (`node:test`)
- **Apache License 2.0** — required for Midnight Buildathon submissions
- **SVG / cairosvg / Python** — branding assets (icon, logo, banner)

## How we built it

1. **Studied the privacy model first.** Read Midnight's Compact reference and the private-guest-list and bulletin-board examples to understand `witness`, `persistentCommit`, `persistentHash`, `disclose()`, sealed ledger fields, and the dual-ledger model.
2. **Designed the privacy boundary before writing code.** Decided up front: confession content never on-chain; identity is a DApp-scoped public key derived from a local secret via `persistentHash` bound to the `"hush:pk:"` namespace (untrackable across DApps); moderator actions prove role via ZK without revealing which moderator.
3. **Wrote the Compact contract** (`contracts/hush.compact`) with 8 circuits, 8 ledger fields, 1 witness, 2 enums. Every `disclose()` call is intentional and documented.
4. **Compiled, hit errors, fixed, recompiled.** The compiler enforced privacy invariants we hadn't fully appreciated (see Challenges). Final result: 8 circuits compile successfully.
5. **Wrote a simulation/smoke test** (`tests/hush.simulation.js`) that validates the source surface area (required circuits, ledger fields, enums, witness, `persistentCommit`/`persistentHash`/`disclose` usage) and documents the lifecycle scenario for the managed API.
6. **Authored README, threat model, slide deck outline, submission checklist.** All Buildathon-required artifacts.
7. **Generated branding assets** (icon, logo, banner) as SVG and exported to PNG.
8. **Licensed under Apache 2.0** per the Buildathon rules; pushed to GitHub with the `midnightntwrk` topic.

## What we learned

- **Midnight's dual-ledger model is the actual innovation.** Privacy isn't bolted on via client-side encryption (the Web2 pattern) — it's enforced at the protocol level. The contract *cannot* accidentally leak private data because the compiler refuses to let witness-derived values cross the ledger boundary without explicit `disclose()`.
- **ZK proofs of role are the missing piece for moderation.** "Accountability without identity exposure" is a phrase that finally clicked: a moderator can flag a confession and prove they were authorized, without anyone learning which moderator they were. This resolves the false dichotomy that has plagued anonymous platforms for two decades.
- **`persistentCommit` and `persistentHash` are the two primitives that make DApp-scoped privacy work.** Commitments bind content to identity without revealing either; persistent hashes bound to a DApp-specific namespace make users untrackable across DApps.
- **Compact's bounded language constraint is a feature.** No recursion, fixed-size types, bounded loops — it forces you to think about state size up front, which keeps proving circuits finite and auditable.
- **Iterating across Waves is the right shape for this kind of work.** Shipping the contract first, frontend second, lets the engineering effort land where the rubric weight is highest (40% Engineering) while preserving a meaningful milestone for Wave 3.

## What's next for Hush

**Wave 3 (immediate):**
- TypeScript SDK integration via `@midnight-ntwrk/sdk`
- Next.js frontend with: confession composer (computes content hash client-side), anonymous reaction buttons, moderator dashboard (proves moderator key locally, no identity reveal)
- Local proof server + Preprod deployment
- Shielded tipping via `nativeToken()` and `sendUnshielded` / `receiveUnshielded`
- Optional encrypted confession content storage using a DApp-scoped symmetric key derived from the user's secret, so confessions can be selectively revealed to specific parties (a therapist, a compliance officer) without going on-chain
- Categorized commitment index for filtered browsing

**Beyond Wave 3:**
- **Cross-DApp reputation proofs** — "I have had N confessions accepted by moderators across Hush and partner DApps" without revealing which confessions or which DApps
- **Selective-disclosure credentials for category-gated posting** — e.g. prove "I am a verified health professional" before posting in the HEALTH category, without revealing identity
- **Cardano identity attestation interop** — leverage Midnight's interoperability story for credential verification across ecosystems
- **Mobile app via Kuira SDK** — privacy-focused mobile distribution
