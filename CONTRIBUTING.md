# Contributing to Supremacy Live!

Two humans (Alex, Matt), each running Claude Code as an orchestrator over the same four agents.
This file is the rulebook every agent and human follows. If reality and this file disagree, fix
the file in a PR — don't quietly work around it.

Contents: 1 Team · 2 Workflow · 3 Branches, commits, PRs · 4 Code rules · 5 Architecture &
layout · 6 Design system · 7 Checks & testing · 8 Deployment pipeline · 9 Docs & knowledge ·
10 OpenFront usage · 11 Definition of done

---

## 1. Team

| Role | Who | Owns | Never does |
|---|---|---|---|
| Human | Alex, Matt | Product calls, plan approval, merge & promote decisions | — |
| Orchestrator | Each human's main Claude Code session | Routing, asking humans, issue hygiene | Writes feature code itself |
| `planner` | `.claude/agents/planner.md` | `PLAN.md` — technical plan + plain-speak summary + open questions | Edits `src/` |
| `builder` | `.claude/agents/builder.md` | Code, tests, `BUILD.md`, commits on the job branch | Guesses past an open question; edits knowledge docs |
| `reviewer` | `.claude/agents/reviewer.md` | `REVIEW.md`, `docs/knowledge/`, ledger, ADRs, CHANGELOG, post-mortem | Fixes game code |
| `deployer` | `.claude/agents/deployer.md` | Push, PR, CI watch, merge (after human yes), deploy watch, promote | Writes code; merges without a human yes |

## 2. Workflow

```
Issue ─► branch ─► PLAN ─► human approves ─► BUILD ─► REVIEW ─(changes)─► BUILD
                                                        │
                                                    APPROVED
                                                        │
                         human: "deploy" ─► PR ─► CI green ─► human: "merge" ─► main ─► staging
                         human: "promote" ─► production workflow ─► GitHub env approval ─► live
```

- **Every job starts from an issue.** One issue = one branch = one PR = one work folder.
- **Work folder:** `docs/work/<issue>-<slug>/` holding `PLAN.md`, `BUILD.md`, `REVIEW.md`, copied
  from `docs/work/_template/`. These stay in the repo after merge — they're our history of why.
- **Human gates:** plan approval, answers to questions, "deploy", "merge", "promote". Agents
  never self-approve a gate.
- **Questions over assumptions.** Planner and builder stop and ask when a decision isn't written
  down. A decision that matters beyond one job goes into `docs/OPEN-QUESTIONS.md` or an ADR.
- **Two machines, one truth.** Anything the other human's Claude needs to know lives in the repo.
- **Claiming work:** assign the issue to yourself + `status:in-progress` before planning. Check
  open issues/PRs in the same `area:` before you start.
- **Review loop cap:** two CHANGES_REQUESTED cycles, then a human looks.

## 3. Branches, commits, PRs

- Branches: `alex/<issue>-<slug>` or `matt/<issue>-<slug>` from fresh `main`.
- `main` is protected: PR required, no force-push, no deletion, `ci` check required (added by #2).
- Commits: [Conventional Commits](https://www.conventionalcommits.org/) —
  `feat|fix|refactor|perf|test|docs|chore|ci(<area>): <imperative summary>`.
  Areas: `sim`, `net`, `ui`, `map`, `data`, `match`, `econ`, `ci`, `docs`.
- PRs are **squash-merged**; PR title becomes the commit on `main`, so it follows the same format
  and ends with `(#<issue>)`.
- PR body follows `.github/pull_request_template.md`.
- Keep PRs small — aim under ~400 changed lines of code (docs/tests don't count against it).
- Rebase on `main` before review if `main` moved. Only ever force-push your own job branch, and
  only with `--force-with-lease`.

## 4. Code rules (Luau)

Language default is **Luau, strict mode** (confirm in Q2).

**Every file**
- `--!strict` on line 1. `any` needs a comment explaining why.
- Modules return one table or one function; no globals; no `_G`/`shared`.
- Names: `PascalCase` modules, types, and classes; `camelCase` functions, locals, fields;
  `SCREAMING_SNAKE` constants. File name = module name.
- No magic numbers for gameplay values — they live in `src/shared/Config/` with a comment on units.
- Prefer small pure functions. Max ~300 lines per module before splitting.
- Comments explain *why*, not *what*. Public functions get a one-line doc comment with types.

**Server authority (non-negotiable)**
- The server owns all game state. Clients send **intents** ("attack tile X with 40%"), never
  outcomes ("I now own tile X").
- Every remote: one definition in `src/shared/Net/`, a typed payload, and a server-side validator
  (type, range, ownership, rate limit). Invalid input is dropped and counted, never trusted.
- No client timestamps, positions, or resource counts are trusted.

**Simulation library (`lib/OpenFrontCore/`)** — our Luau port of OpenFront's core
- Follow `lib/OpenFrontCore/README.md` — it's the porting rulebook (structure mirroring,
  attribution header, TS→Luau gotchas, golden tests, upstream sync).
- Pure Luau. **No Roblox APIs** (`game`, `Instance`, services, `task`, `workspace`) so it runs
  headless in tests.
- Deterministic: the library's seeded PRNG only; no `os.time`/`os.clock`/`math.random`; no logic
  that depends on `pairs()` ordering.
- Fixed tick. Each tick is `state + intents → new state + events`.
- Hot paths avoid per-tick table allocations; use `buffer` for the tile grid.
- The game never reaches into library internals; it uses the library's public API. Game-specific
  behavior goes through config or injection points, not edits to ported logic.

**Roblox side**
- Clean up every connection (use a maid/trove pattern). No leaked `RBXScriptConnection`s.
- No yielding inside the tick loop.
- Persistence goes through one data module with session locking and retries; never call
  DataStores directly from feature code.
- Assets (images, sounds) are referenced by ID from `src/shared/Config/Assets.luau`, never inline.
- Nothing is authored only in Studio: the place is built from this repo by Rojo. If it isn't in
  git, it doesn't exist.
- Don't author instance types the Open Cloud publish API can't update (EditableImage,
  EditableMesh, PartOperation, SurfaceAppearance, BaseWrap) into the place file — create them at
  runtime if needed.

## 5. Architecture & layout

Finalized by issue #2 (scaffold). Target layout:

```
lib/
  OpenFrontCore/   → ReplicatedStorage.OpenFrontCore — our Luau port of OpenFront src/core
                     (pure, deterministic, mirrors upstream file layout; AGPL + attribution)
src/
  shared/          → ReplicatedStorage.Shared
    Config/        balance values, map list, asset IDs, feature flags, source/credits link
    Net/           remote definitions, payload types, validators
    Types/         shared type definitions
  server/          → ServerScriptService.Server
    Match/         match lifecycle, tick loop, feeds validated intents into OpenFrontCore
    Services/      matchmaking, ranked, leaderboards, economy (meta), monetization
    Data/          profiles & persistence (single module)
  client/          → StarterPlayer.StarterPlayerScripts.Client
    UI/
      Theme/       design tokens (the only place raw style values may exist)
      Components/  reusable UI building blocks
      Screens/     full screens composed from Components
    Map/           map renderer + input (pan/zoom/select)
    Controllers/   client state, remote listeners
tests/             headless tests: OpenFrontCore/ (mirrors lib), shared/ (run in CI under Lune)
scripts/           check, build, golden-test, and tooling scripts
docs/              product, knowledge, decisions, work folders, OpenFront ledger
```

- Toolchain pinned with **Rokit** (`rokit.toml`): Rojo, Wally, StyLua, Selene, luau-lsp, Lune
  (exact versions chosen and pinned in #2). Packages via **Wally**.
- Place topology (single place vs lobby + match places) — see Q8.
- Networking model (how the tile map replicates) — see Q9 and spike #4.

## 6. Design system

The game is screens-only, so the UI *is* the game. Consistency is enforced, not hoped for.

**Tokens — `src/client/UI/Theme/`** (the only place raw style values may appear)
- `Colors` — semantic names (`surface`, `surfaceRaised`, `textPrimary`, `accent`, `danger`,
  `success`, `warning`), not hex names. Player/territory colors come from a generated,
  colorblind-checked palette, not hand-picked per feature.
- `Typography` — font family, weights, and a size scale (`xs…xxl`).
- `Spacing` — a fixed scale (e.g. 4/8/12/16/24/32 px before scaling).
- `Radius`, `Stroke`, `Elevation`, `ZIndex` layers (map < HUD < panels < modals < toasts).
- `Motion` — durations and easing styles for every tween.
- `Icons` — one icon set, referenced by name.

**Components — `src/client/UI/Components/`**
Button, IconButton, Panel, Modal, Tabs, List, Tooltip, Toast, ProgressBar, ResourceCounter,
PlayerChip, Badge, Slider… Each component documents its props and states (default, hover,
pressed, disabled, focused) in `docs/knowledge/ui-components.md`.

**Rules**
- Screens compose components. Screens don't style raw Frames.
- No `Color3.fromRGB`, `Color3.new`, `Color3.fromHex`, `Font.new`, `Enum.Font.*`, or literal
  text sizes outside `UI/Theme/`. CI fails if found (`scripts/check-design-system`, built in #2).
- **Desktop-first (ADR 0002).** Design at **1920×1080**. Every screen must stay usable at
  1366×768, look right at 2560×1440, and not stretch panels on ultrawide. Use a single UIScale
  driven by viewport height against the 1080 reference, plus anchored layouts.
- Mouse + keyboard is the input model: every interactive element has hover and pressed states,
  information-dense items get tooltips, the map supports scroll-to-zoom, drag-to-pan, and
  right-click context actions, and frequent actions get hotkeys (one central keybind table).
- Touch, gamepad, and phone layouts are out of scope until a new ADR says otherwise.
- All player-facing strings go through one localization table — no hard-coded strings in screens.
- New component or token → reviewer updates `docs/knowledge/ui-components.md` / the theme doc.
- UI library (React-Lua, Fusion, or Vide) — see Q3.

## 7. Checks & testing

One command runs everything locally and in CI (created in #2): `scripts/check` →
1. **Format** — `stylua --check src tests`
2. **Lint** — `selene src tests`
3. **Types** — `luau-lsp analyze` with Roblox definitions
4. **Design system** — `scripts/check-design-system`
5. **Tests** — headless tests run with Lune: `OpenFrontCore` unit + golden + determinism tests,
   and `src/shared/` (Net validators, Config sanity). Also a header check: every file in
   `lib/OpenFrontCore/` carries the attribution header.
6. **Build** — `rojo build -o build/place.rbxl` succeeds

Rules: every ported module ships with tests (golden tests against upstream wherever feasible). Bug fixes ship with a test that would have caught the
bug. Agents paste real check output into `BUILD.md`/`REVIEW.md` — never "should pass".
Play-test steps in Studio are listed in `BUILD.md` for anything a test can't cover.

## 8. Deployment pipeline

Two Roblox experiences, both owned by the game's Roblox Community (see `docs/SETUP-MATT.md`):
- **Staging** — private, for us and testers. Separate experience so its DataStores never touch
  real player data.
- **Production** — public.

GitHub Actions (built in #2):

| Workflow | Trigger | Does |
|---|---|---|
| `ci.yml` (check name **`ci`**) | every PR, push to `main` | `scripts/check`; uploads `place.rbxl` artifact |
| `deploy-staging.yml` | push to `main` (i.e. every merge) | build → publish to **staging** via Open Cloud → write place version to job summary |
| `deploy-production.yml` | manual `workflow_dispatch` | uses GitHub environment `production` (requires Alex or Matt to approve) → **license gate** (fails unless `Config` has a non-empty public source URL and the attribution notice is present in the loading screen + Credits) → build from `main` → publish to **production** → tag `vYYYY.MM.DD-N` → move `CHANGELOG` Unreleased under the tag |

Publishing uses Roblox's Open Cloud place-publishing API:
`POST https://apis.roblox.com/universes/v1/{universeId}/places/{placeId}/versions?versionType=Published`
with header `x-api-key` and the `.rbxl` as the body.

Secrets & variables (per GitHub environment, never in the repo):
- `ROBLOX_API_KEY` (secret) — Open Cloud key with **universe-places → Write** on that experience.
- `ROBLOX_UNIVERSE_ID`, `ROBLOX_PLACE_ID` (variables). Multiple places → one ID per place.

Rollback: re-run `deploy-production.yml` on the previous tag, or restore a prior version from the
Creator Dashboard version history.

## 9. Docs & knowledge

| Where | What | Who writes |
|---|---|---|
| `docs/PRODUCT.md` | Vision, pillars, scope | Humans (via a docs PR) |
| `docs/OPEN-QUESTIONS.md` | Undecided things, options, decision once made | Orchestrator, on human answer |
| `docs/decisions/NNNN-*.md` | ADRs — lasting decisions and why | Reviewer (or orchestrator) |
| `docs/knowledge/*.md` | How each of our systems works today | Reviewer |
| `docs/openfront/README.md` | License gate, pinned upstream, port ledger | Reviewer |
| `docs/work/<issue>-<slug>/` | Per-job plan, build notes, review + post-mortem | Planner, builder, reviewer |
| `CHANGELOG.md` | Player-facing changes | Reviewer (Unreleased), deployer (on promote) |

Knowledge pages are written for a future agent with zero context: purpose, how it works, key
files, data/remotes, config knobs, gotchas.

## 10. OpenFront usage

- OpenFront (https://github.com/openfrontio/OpenFrontIO) is a TypeScript browser game, not a
  Roblox library — so **we build the library**: `lib/OpenFrontCore/`, a faithful Luau port of
  upstream `src/core`. Rules in `lib/OpenFrontCore/README.md`.
- Upstream is the source of truth for any gameplay system **until** we've ported/documented it —
  after that, our library and `docs/knowledge/` win, and upstream changes are opt-in syncs.
- Read upstream from `../openfront-upstream` at the SHA pinned in `docs/openfront/README.md`.
  Never vendor the upstream repo into ours; only ported modules live here.
- **License (ADR 0001):** the game is AGPL-3.0 with OpenFront's additional attribution terms.
  Ported files carry the header; the game shows the notices and a source link. Upstream assets
  need CC BY-SA attribution. Never use OpenFront's name/logo as our branding.
- RoWorld (https://www.roblox.com/games/137151641329864/RoWorld) is a **feel/UX reference only**.

## 11. Definition of done

A job is done when: acceptance criteria met · all §7 checks green locally and in CI · reviewer
APPROVED · knowledge/ledger/CHANGELOG updated · PR squash-merged after a human yes · staging
deploy succeeded · issue closed with the staging version noted.
