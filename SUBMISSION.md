# Wave 2 Submission Checklist

Use this checklist when submitting to AKINDO. Every item maps to a
Buildathon requirement.

## Required artifacts (all must be present)

- [ ] Public GitHub repository (this repo) pushed to GitHub
- [ ] `midnightntwrk` topic added to the GitHub repo
    (Repo → About → Topics → add `midnightntwrk`)
- [ ] Apache 2.0 LICENSE file at repo root  → `./LICENSE` ✅
- [ ] README with project, setup, architecture, Midnight integration,
      and how judges can test  → `./README.md` ✅
- [ ] At least one Compact contract that compiles successfully
      → `./contracts/hush.compact` (run `npm run compact`)
- [ ] Slide deck (pitch presentation)  → `./docs/PITCH.md` ✅
- [ ] Demo / video pitch (record separately; see script in PITCH.md)
- [ ] Description of progress completed during Wave 2
- [ ] If resubmitting from Wave 1: explanation of what changed
    (N/A for this submission — first Wave entry)

## Technical gate (auto-DQ if not met)

- [ ] `npm run compact` exits 0 and emits `src/managed/hush/`
- [ ] Repository is publicly accessible
- [ ] Repository has the `midnightntwrk` GitHub topic label

## Suggested pre-submission commands

```bash
npm install
npm run lint        # compact check
npm run compact     # compile the contract
npm test            # run the smoke + simulation suite
```

All three must succeed before submitting.

## AKINDO submission fields (suggested content)

- **Project name:** Hush
- **Tagline:** A privacy-first anonymous confession platform on Midnight.
- **One-paragraph summary:** Hush lets people post anonymous confessions,
  react to others, and have those confessions moderated — all on Midnight,
  where zero-knowledge proofs make it possible to verify identity,
  ownership, and authorization without ever revealing who anyone is or what
  they confessed. The Wave 2 submission ships a compiling Compact contract
  with private state via witnesses, DApp-scoped public keys, persistent
  content commitments, and ZK-moderated flagging. Wave 3 will add the
  TypeScript SDK and a Next.js frontend.
- **GitHub URL:** _your repo URL_
- **Slide deck URL:** _link to `docs/PITCH.md` in the repo, or uploaded file_
- **Demo video URL:** _link to your recorded video_
- **Wave 2 progress description:**
    "Designed and implemented the Hush Compact contract covering the full
    anonymous confession lifecycle: post, react, take-down, moderator
    flag/unflag, and public traction counters. Privacy design uses
    persistentCommit for content (content never on-chain), persistentHash
    for DApp-scoped public keys (untrackable across DApps), and a
    localSecretKey witness for private state. Wrote a source smoke test
    and a managed API simulation. Authored Apache 2.0 license, README with
    threat model, and slide deck. Frontend and TypeScript SDK integration
    are scheduled for Wave 3."
- **License:** Apache 2.0
