# OpenFrontCore — our Luau port of OpenFront's simulation

A pure-Luau, deterministic port of upstream OpenFront's `src/core` at the SHA pinned in
`docs/openfront/README.md`. It's the game's simulation engine. The game (`src/`) depends on it;
it depends on nothing in the game. Licensed AGPL-3.0 with OpenFront's additional terms (ADR 0001).

Rojo maps this folder to `ReplicatedStorage.OpenFrontCore` (set up in #2).

## Design rules

1. **Faithful first.** Port behavior exactly as upstream does it, including numbers and edge
   cases. Improve later, on purpose, as a recorded divergence.
2. **Mirror upstream's structure.** One upstream TS file → one Luau module with the same name,
   in the same relative folder. This makes upstream syncs a file-by-file diff instead of archaeology.
   Upstream `src/core/game/Foo.ts` → `lib/OpenFrontCore/game/Foo.luau`.
3. **Pure and headless.** No Roblox APIs (`game`, `Instance`, services, `task`, `workspace`,
   `os.time`, `tick`). Runs under Lune in CI.
4. **Deterministic.** Same seed + same intents → same state, every time, every machine.
5. **Game-specific code stays out.** Roblox networking, UI, persistence, monetization, ranked live
   in `src/`. If the game needs different behavior, add a config value or an injection point
   here — don't fork logic inside the library.
6. **Every file has the attribution header:**
   ```lua
   --!strict
   -- Derived from OpenFront (https://github.com/openfrontio/OpenFrontIO) @ <pinned SHA>, <upstream path>
   -- © OpenFront and Contributors. Licensed under AGPL-3.0 with additional terms; see LICENSE.
   ```
7. **Divergences are recorded.** Any intentional difference from upstream gets a
   `-- DIVERGENCE: <why> (ADR/issue)` comment at the spot *and* a ledger row marked `diverged`.

## TypeScript → Luau porting gotchas (check every module)

| JS/TS | Luau | What to do |
|---|---|---|
| Arrays are 0-based | Tables are 1-based | Pick one convention per module and comment it. For tile indices that are *math* (x + y·width), keep 0-based numbers and store in buffers/offset arrays; never mix silently. |
| `Map`/`Set` iterate in insertion order | `pairs()` order is unspecified | Any loop whose order affects results must use an ordered structure (array of keys, or an ordered-map helper in `util/`). Non-negotiable for determinism. |
| Bitwise ops are **signed** 32-bit (`\|`, `&`, `>>`, `<<`) and `>>>` is unsigned | `bit32` is **unsigned** 32-bit | Port bit tricks through helpers in `util/Bits.luau` that reproduce JS semantics; unit-test them against known JS outputs. |
| `Math.floor`, `%` on negatives | `%` in Luau is floored (JS `%` is truncated) | Use a `jsMod` helper where upstream relies on `%` with possible negatives. |
| Integer division via `\| 0` | `//` floors, doesn't truncate | Use `math.floor`/a `trunc` helper matching JS. |
| Numbers are doubles | Numbers are doubles | Good news: arithmetic matches. Watch for upstream `BigInt` (port explicitly if present). |
| `Math.random()` | — | Must go through the library's seeded PRNG — port upstream's PRNG exactly. |
| Classes, inheritance, `this` | Metatables | Use one class pattern library-wide (defined in #3) with typed `self`. |
| `undefined` vs `null` | only `nil` | Where upstream distinguishes them, model it explicitly. |
| Strings are UTF-16, `.length` | `#` is bytes | Avoid string-length logic in sim; use `utf8.len` if unavoidable. |
| Typed arrays (`Uint8Array`, `Uint16Array`) | `buffer` | Use `buffer` for the tile grid and other big numeric arrays. |
| npm dependencies | — | List each one in spike #3; port the minimum needed into `util/` with its own license noted. |

## Testing

- Each ported module gets tests in `tests/OpenFrontCore/` mirroring its path.
- **Golden tests:** where possible, record outputs from upstream for fixed seeds and inputs
  (a small Node script in `scripts/golden/` run against `../openfront-upstream`) and assert our
  port produces identical results. This is the strongest guarantee the port is faithful.
- Determinism test: run the same seeded scenario twice and compare a state hash.

## Porting order (proposed, refined by spike #3)
PRNG & utils → map/terrain data → tile ownership & borders → spawn → troops/population →
attacks/expansion → gold economy → structures → alliances → bots → win conditions →
naval/trade → missiles (post-MVP).

## Syncing with upstream
1. Fetch upstream, pick a new SHA, list changed files under `src/core` since the pinned SHA.
2. For each changed file that maps to a ported module, re-port the change (or record why we skip it).
3. Re-run golden tests against the new SHA.
4. Bump the pin + headers in one PR labelled `area:sim`, `type:chore`.
