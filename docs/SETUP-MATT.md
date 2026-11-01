# Setup — Matt (and any future contributor)

You'll run your own Claude Code locally on this repo. It acts as your orchestrator and runs the
same four agents Alex's does (planner, builder, reviewer, deployer). The repo is how your two
Claudes stay in sync.

## 1. Accept access
- Accept the GitHub invite to `Synxpsis/of-supremacy-rblx` (email or https://github.com/notifications).
- Accept the Roblox Community invite when Alex sends it (see "Roblox ownership" below).

## 2. Install (once)
| Tool | Why | Check |
|---|---|---|
| Git | version control | `git --version` |
| GitHub CLI `gh` | agents open PRs/issues | `gh --version` |
| Claude Code | your orchestrator | `claude --version` |
| Rokit | installs the pinned Roblox toolchain (Rojo, StyLua, Selene, luau-lsp, Lune, Wally) — https://github.com/rojo-rbx/rokit | `rokit --version` |
| Roblox Studio | play-testing | — |
| Rojo Studio plugin | syncs the repo into Studio | — |

Then sign in to GitHub from the terminal with the scopes the agents need:
```
gh auth login
gh auth refresh -s workflow
```

## 3. Clone and start
```
git clone https://github.com/Synxpsis/of-supremacy-rblx.git
cd of-supremacy-rblx
claude
```
Starting `claude` **inside the repo folder** matters: that's how it loads `CLAUDE.md` and the
agents in `.claude/agents/`.

## 4. First message to your Claude Code
Paste this:

> I'm Matt. This is my first session on this repo. Read CLAUDE.md, CONTRIBUTING.md,
> docs/PRODUCT.md and docs/OPEN-QUESTIONS.md, then run the first-run checklist from CLAUDE.md.
> Set up the OpenFront upstream reference clone next to the repo if it's missing. Tell me what's
> ready, what's blocking, and which issue we should pick up first. Don't start any work yet.

## 5. Day to day
- Start every session in the repo folder with `claude`. It pulls `main` and checks what Alex is
  working on before you start.
- Pick or create an issue → planner writes a plan → you approve → builder builds → reviewer
  reviews and documents → you say **"deploy"** → deployer opens the PR → you say **"merge"**.
- Anything decided in your chat that Alex's Claude needs to know must land in the repo
  (`docs/OPEN-QUESTIONS.md` or `docs/decisions/`). Your Claude does this; just confirm.
- Studio: open the staging place, connect the Rojo plugin, and run `rojo serve` from the repo to
  play-test a branch live.

## Roblox ownership (Alex sets up, Matt joins)
1. Create a **Roblox Community** for the game (Q4). Give Matt a role with experience edit access.
2. Under the Community, create two experiences: **Supremacy Live! (Staging)** — private — and
   **Supremacy Live!** — production.
3. Set max players per server to fit the 12–24 player design (exact number decided in planning).
   Under **Playable Devices**, enable **Computer** only (ADR 0002 — desktop-first) on both
   experiences.
4. Create an **Open Cloud API key** (Creator Dashboard → Credentials) owned by the Community with
   `universe-places` **Write** on both experiences. Allow GitHub Actions' IP ranges (or allow any IP).
5. In GitHub → Settings → Environments, add per environment (`staging`, `production`):
   secret `ROBLOX_API_KEY`, variables `ROBLOX_UNIVERSE_ID` and `ROBLOX_PLACE_ID`.
   Never paste the key into chat or commit it.

These are needed by the time issue #2 (scaffold + deploy pipeline) is built.
