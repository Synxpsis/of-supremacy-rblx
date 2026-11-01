# 0001 — Port OpenFront's core under AGPL-3.0

- Status: accepted
- Date: 2026-10-31
- Deciders: Alex, Matt
- Related: Q1, Q2, Q15 in docs/OPEN-QUESTIONS.md

## Context
OpenFront is the gameplay source of truth, but it's a TypeScript browser game — there is no
Roblox/Luau library to depend on. Its code is AGPL-3.0 with additional attribution terms, its
assets are CC BY-SA 4.0, and its brand/logos (`proprietary/` upstream) are not licensed.

## Options
1. **Port** — translate upstream's deterministic core into Luau; whole game becomes AGPL-3.0.
2. **Clean-room** — rebuild behavior from the wiki only; repo can stay closed; slower, riskier
   fidelity, blurry line in practice.

## Decision
**Port.** We build and maintain our own library, `lib/OpenFrontCore/`, a faithful Luau port of
upstream's core at a pinned SHA, and license the game under AGPL-3.0 with OpenFront's additional
terms. Revenue (cosmetics, passes, private servers) is compatible with this.

## Consequences
- Repo root carries upstream's `LICENSE` (incl. additional terms) and `LICENSING.md`;
  `LICENSE-ASSETS` too if we ever use upstream assets.
- Every ported file carries the attribution header from `docs/openfront/README.md`.
- The game shows "© OpenFront and Contributors" on the loading screen and in a Credits screen,
  plus a link to our source (Q15) — a production-release blocker.
- OpenFront's name and logos are never our branding.
- Porting is a first-class workstream: ledger, porting guide, upstream sync process
  (`lib/OpenFrontCore/README.md`).
- Not legal advice; revisit if the business model changes.
