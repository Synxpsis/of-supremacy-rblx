# OpenFront — source of truth, license gate, port ledger

OpenFront (https://github.com/openfrontio/OpenFrontIO, game at https://openfront.io, wiki at
https://openfront.wiki) is a TypeScript browser RTS. Its deterministic simulation lives in
upstream `src/core`; client in `src/client`; server in `src/server`; assets in `resources/`;
brand/logos in `proprietary/`.

It is our **source of truth for gameplay behavior until we document a system ourselves** in
`docs/knowledge/`. After that, our docs win and upstream changes are opt-in.

## Pinned upstream
| Field | Value |
|---|---|
| Repo | https://github.com/openfrontio/OpenFrontIO |
| Local path | `../openfront-upstream` (sibling of this repo — never inside it) |
| Pinned SHA | `030274df6e377a57e784a9f6f9865720b0dfbbba` |
| Pinned on | `2026-10-31` |

Bump the pin only in a dedicated PR that lists what changed upstream in systems we track.

## License gate
- Code: **AGPL-3.0 + additional attribution terms** (keep "© OpenFront and Contributors"
  notices in visible places like the footer/loading screen in modified versions).
- Assets (`resources/`): **CC BY-SA 4.0** (attribution + share-alike).
- Brand (`proprietary/`): not for reuse. Never.

**Decided: Port** (Q1, ADR 0001). That means:
- ✅ Translate upstream `src/core` into Luau — **only into `lib/OpenFrontCore/`**, following
  `lib/OpenFrontCore/README.md` (structure mirroring, porting gotchas, golden tests).
- ✅ Every ported file starts with the attribution header (see the library README).
- ✅ Our repo carries upstream's `LICENSE` and `LICENSING.md` at the root.
- ✅ The game shows "© OpenFront and Contributors" on the loading screen and in Credits, plus a
  link to our source (Q15) — required before production is public.
- ⚠️ Upstream `resources/` assets (maps, images) may be used only with CC BY-SA attribution;
  add `LICENSE-ASSETS` and a credits entry the first time we do.
- ❌ Never use anything from `proprietary/` or OpenFront's name/logo as our branding.
- ❌ Don't port upstream client/server code (`src/client`, `src/server`) — Roblox replaces those.
  Read them for behavior only.

## Port ledger
Status: `upstream-only` → `documented` (our spec exists) → `ported` (implemented in
`lib/OpenFrontCore/`, matches upstream, golden tests pass) or `diverged` (implemented,
intentionally different — say why). Spike #3 fills in the upstream paths and splits/merges rows
to match upstream's real structure.

| System | Upstream path(s) | Status | Our doc | Notes |
|---|---|---|---|---|
| Map loading & terrain (land/water/terrain types) | TBD | upstream-only | — | |
| Tile ownership & territory borders | TBD | upstream-only | — | |
| Spawn selection | TBD | upstream-only | — | |
| Attacks & expansion (troop flow into tiles) | TBD | upstream-only | — | |
| Population / troop growth | TBD | upstream-only | — | |
| Gold economy | TBD | upstream-only | — | |
| Structures (cities, ports, defenses, silos, SAM…) | TBD | upstream-only | — | |
| Naval & trade units | TBD | upstream-only | — | |
| Missiles / nukes | TBD | upstream-only | — | post-MVP |
| Alliances & diplomacy | TBD | upstream-only | — | |
| Bots / AI nations | TBD | upstream-only | — | |
| Win conditions & match end | TBD | upstream-only | — | |
| Game config & balance values | TBD | upstream-only | — | |
| PRNG & determinism utilities | TBD | upstream-only | — | |
| Game modes (FFA, teams) | TBD | upstream-only | — | |
