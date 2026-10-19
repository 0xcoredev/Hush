# Hush — Wave 2 Pitch Deck Outline

> Slide deck outline for the Midnight Buildathon Wave 2 submission.
> Record a 3–5 minute video pitch walking through these slides.

## Slide 1 — Title

**Hush**
A privacy-first anonymous confession platform on Midnight.

- Built for the Midnight Buildathon — Wave 2
- License: Apache 2.0
- Repo: _your GitHub URL here_ (tagged `midnightntwrk`)

## Slide 2 — The problem

Anonymous confession platforms have historically failed in one of two ways:

1. **Pseudonymous-but-tracked** — the platform stores identifiers that can be
   subpoenaed, leaked, or correlated across sessions. "Anonymous" is a lie.
2. **Fully anonymous-but-unmoderatable** — no way to remove abuse, spam, or
   harm without a central operator that re-identifies users.

The result: users either don't trust the platform, or the platform becomes
unusable. Existing Web2 confession apps (and most Web3 attempts) sit on one
end of this spectrum.

## Slide 3 — Why Midnight

Midnight's dual-ledger model resolves the tension.

- Public ledger state (commitments, counters, flagged sets) is verifiable by
  anyone.
- Zero-knowledge circuits prove authorization locally — the user's secret
  never leaves their device.

Hush enforces rules ("only the original poster can take down", "only a
moderator can flag", "one reaction per user per confession") **without
collecting any personally identifying data**.

## Slide 4 — How Hush uses Midnight (capability map)

| Midnight capability | Hush usage |
|---|---|
| Compact contracts | Single contract implements the full confession lifecycle. |
| Private state via witnesses | `localSecretKey()` never disclosed. |
| DApp-scoped public keys | `persistentHash` over `"hush:pk:"` + secret — untrackable across DApps. |
| Persistent commitments | `persistentCommit<Bytes<32>>(contentHash, posterKey)` proves a confession exists without storing content. |
| Dual-ledger model | Public commitments + counters only; content lives off-chain. |
| Explicit disclosure | Every privacy-boundary crossing is auditable in source. |
| Sealed ledger fields | Moderator key commitment cannot be reassigned. |
| ZK proofs of role | Moderators flag by proving they hold the DApp key — accountability without identity. |

## Slide 5 — Contract design

File: `contracts/hush.compact`

- 9 circuits: `confess`, `react`, `takeDown`, `flag`, `unflag`,
  `confessionCount`, `reactionCount`, `flagCount`, `getDappPublicKey`
- 8 ledger fields, all commitment sets or counters
- 1 witness: `localSecretKey(): Bytes<32>`
- 2 enums: `ConfessionState`, `Category`

**Privacy invariant:** the only values that ever cross `disclose()` are
already-hash-derived commitments and public counters. Confession content and
user identity never enter a proof.

## Slide 6 — Lifecycle walkthrough

1. **Post** — User computes `contentHash` off-chain, calls
   `confess(contentHash, category)`. Contract stores
   `persistentCommit(contentHash, posterKey)`; content never touches chain.
2. **React** — Different user calls `react(confessionCommit)`. Contract
   stores `persistentCommit(confessionCommit, reactorKey)`; double-react
   rejected via set membership.
3. **Moderate** — Moderator calls `flag(confessionCommit)`. Contract checks
   `moderator == getDappPublicKey(localSecretKey())` in-ZK. No one learns
   which moderator acted.
4. **Take down** — Original poster calls `takeDown(contentHash)`. Contract
   regenerates the commitment from their secret; ownership proven without
   identity disclosure. Commitment moves to `takenDownCommits`.

## Slide 7 — What ships in Wave 2

- ✅ Compiling Compact contract (`npm run compact`)
- ✅ Source smoke test + managed API simulation (`npm test`)
- ✅ Apache 2.0 license, `midnightntwrk` GitHub topic
- ✅ README with privacy design and threat model
- ✅ This slide deck + demo video

## Slide 8 — Wave 3 roadmap

- TypeScript SDK integration (`@midnight-ntwrk/sdk`)
- Next.js frontend: confession composer, anonymous reactions, moderator
  dashboard
- Local proof server + Preprod deployment
- Shielded tipping via `nativeToken()`
- Optional encrypted content storage for selective reveal

## Slide 9 — Beyond Wave 3

- Cross-DApp reputation proofs ("N confessions accepted across Hush +
  partner DApps" without revealing which)
- Selective-disclosure credentials for category-gated posting (e.g. verified
  health professional before posting in HEALTH)
- Cardano identity attestation interop

## Slide 10 — Team & repo

- Repository: _your GitHub URL_
- GitHub topic: `midnightntwrk`
- License: Apache 2.0
- Contact: _your Discord / AKINDO handle_

---

## Demo video script (3–5 minutes)

1. **(0:00–0:30)** Title slide + one-sentence pitch.
2. **(0:30–1:00)** The problem slide. Emphasize the false dichotomy
   between pseudonymous-tracked and anonymous-unmoderatable.
3. **(1:00–1:45)** Why Midnight. Highlight dual-ledger + ZK proofs of
   authorization.
4. **(1:45–2:30)** Open `contracts/hush.compact` in the editor. Walk
   through `confess`, `react`, `flag`, `takeDown`. Point at every
   `persistentCommit` / `persistentHash` / `disclose` and explain what
   stays private.
5. **(2:30–3:15)** Run `npm run compact`. Show the compiler output and the
   managed API emitted to `src/managed/hush/`.
6. **(3:15–3:45)** Run `npm test`. Show the smoke test passing and the
   lifecycle scenario output.
7. **(3:45–4:15)** Wave 3 roadmap slide. State clearly what is coming next
   and why it constitutes meaningful new progress.
8. **(4:15–4:30)** Repo URL + `midnightntwrk` topic + license.
