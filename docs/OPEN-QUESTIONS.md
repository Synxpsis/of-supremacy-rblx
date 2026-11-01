# Open questions

Decisions not yet made. **Blocking** means no job in that area should start until it's decided.
When a question is answered: fill in *Decision*, *Decided by*, *Date*, set *Status: decided*,
and write an ADR in `docs/decisions/` if it's lasting. Don't delete answered questions.

---

### Q1 — OpenFront license path · Status: **decided — A. Port** (see ADR 0001)
OpenFront's source is **AGPL-3.0 with additional attribution terms** (modified versions must keep
"© OpenFront and Contributors" notices visible, e.g. footer/loading screen). Its assets are
**CC BY-SA 4.0**, and its logos/brand (`proprietary/` upstream) are not licensed for reuse.
Translating its TypeScript into Luau is a derivative work.

- **A. Port.** Translate OpenFront's `core` into Luau. Our game code is then AGPL-3.0: the repo
  (or the source of each released version) must be made available to players, e.g. a public repo
  linked from an in-game credits screen, plus the attribution notices. Fastest, most faithful.
  Monetization is still possible; the code just can't be closed.
- **B. Clean-room.** Use OpenFront only as a *behavior* reference (wiki, playing the game,
  documented numbers). Write original Luau from our own specs; agents don't read upstream source
  while writing code. Repo can stay private. Slower, needs discipline, and the line gets blurry
  if anyone reads the code.
- **C. Hybrid.** Clean-room by default; port specific modules under AGPL only if we accept A's
  obligations for the whole game (in practice this collapses into A).

Decision: **A. Port.** We accept AGPL-3.0 + OpenFront's attribution terms for the whole game and
build our own Luau port of OpenFront's core as an in-repo library (`lib/OpenFrontCore/`).
Revenue is still on the table. · Decided by: Alex (with Matt) · Date: 2026-10-31

### Q2 — Language · **Blocking for #2** · Status: open
Now that we're porting OpenFront's core faithfully, this is a real trade-off:
- **Luau strict**: native, best tooling and docs, no compile step; the port is a hand
  translation, module by module.
- **roblox-ts**: TypeScript compiled to Luau. The port reads much closer to upstream and future
  upstream syncs are easier to diff. But OpenFront's core may use JS/npm features that roblox-ts
  can't compile, and debugging happens through generated Luau.
Recommendation: settle it in spike #3 by porting the **same small upstream module** both ways
and comparing effort, readability, and how cleanly it diffs against upstream. Default if no
spike: Luau strict.

Decision: — · Decided by: — · Date: —

### Q3 — UI library · **Blocking for UI work** · Status: open
- **React-Lua** (React model; what Roblox itself uses for its app UI)
- **Fusion** (reactive state, popular in the community)
- **Vide** (small, fast, reactive)
Recommendation: spike in #2 — build the same Button + Panel in the top two, pick one.
Decision: — · Decided by: — · Date: —

### Q4 — Who owns the Roblox experiences · Status: open
Recommendation: a **Roblox Community (group)** owns both experiences, Alex and Matt are roles in it,
and the Open Cloud API key is created under it. Avoids a single-person dependency and simplifies
revenue split.
Decision: — · Decided by: — · Date: —

### Q5 — Staging setup · Status: open (recommended default applied)
Recommendation: separate **private staging experience** (own DataStores) rather than a second
place in the production experience.
Decision: — · Decided by: — · Date: —

### Q6 — Do PRs need the other human's approval? · Status: open
Current setting: no required approvals; the session's human confirms merges.
Option: require the other human's review only for `area:sim`, `area:net`, `area:data`, or
changes to `CONTRIBUTING.md`/`CLAUDE.md`/agents (via CODEOWNERS).
Decision: — · Decided by: — · Date: —

### Q7 — Production release cadence · Status: open (recommended default applied)
Recommendation: manual promotion (`deploy-production.yml` + environment approval), batched,
never automatic from `main`.
Decision: — · Decided by: — · Date: —

### Q8 — Place topology · **Blocking for matchmaking work** · Status: open
- **Single place** (lobby phase then match phase in the same server) — simplest for MVP.
- **Lobby place + match place** with reserved servers via TeleportService — standard for
  matchmaking/ranked at scale.
Recommendation: single place for MVP, designed so the match module can move into its own place later.
Decision: — · Decided by: — · Date: —

### Q9 — Network model · **Blocking for Sim/Net work** · Status: open
OpenFront runs lockstep: every client runs the deterministic sim, the server relays intents.
On Roblox, the norm is server-authoritative.
- **Server-authoritative sim + delta replication** of tile ownership (compact buffers), clients
  render only (recommended — cheat-resistant, simpler).
- **Lockstep-style**: server relays intents, clients simulate; less bandwidth, more desync and
  exploit risk.
Tick rate and bandwidth budget decided with spike #4.
Decision: — · Decided by: — · Date: —

### Q10 — Map rendering · Status: open (spike #4)
How to draw a large tile/territory map in GUI on Roblox at good frame rates on desktop
(1920×1080 reference) with smooth pan/zoom (runtime-generated images vs. frame/tile approaches,
chunking, update throttling).
Decision: — · Decided by: — · Date: —

### Q11 — Ranked rating system · Status: open (post-MVP)
Options: Elo variant, Glicko-2, TrueSkill-style for multiplayer FFA. Visible tiers on top of
hidden rating either way.
Decision: — · Decided by: — · Date: —

### Q12 — Game name · Status: open
Working title: Supremacy Live!.
Decision: — · Decided by: — · Date: —

### Q13 — Monetization specifics · Status: open (post-MVP)
Within the guardrails in `docs/PRODUCT.md`: which cosmetics, season pass yes/no, private
server price, starter packs.
Decision: — · Decided by: — · Date: —

### Q14 — Target platform · Status: **decided — desktop-first** (see ADR 0002)
Decision: Desktop (mouse + keyboard) is the target, designed at 1920×1080. Mobile, tablet, and
console are not supported at MVP. · Decided by: Alex · Date: 2026-10-31

### Q15 — Source availability · Status: open (must be decided before production is public)
AGPL requires players to be able to get the source of the version they're playing.
- **Public GitHub repo** linked from an in-game Credits screen (simplest).
- **Private repo + public source mirror** updated on every production release.
Recommendation: develop private now; make the repo public (or start the mirror) no later than the
first public production release. The deploy pipeline enforces a source link in Config.
Decision: — · Decided by: — · Date: —
