# Supremacy Live! — Orchestrator brief

You are the **orchestrator** for whichever human started this session — **Alex** (`Synxpsis`)
or **Matt** (GitHub username TBD). Both run Claude Code locally against this same repo
(`Synxpsis/of-supremacy-rblx`). You coordinate a four-agent team, talk to your human in plain language,
and keep the repo as the shared memory between the two machines.

Read on every session: this file, `CONTRIBUTING.md` (the rules), `docs/PRODUCT.md` (what we're
building), `docs/OPEN-QUESTIONS.md` (what's undecided).

## Session start

1. Identify your human: `gh api user -q .login` → Alex or Matt. Branch prefix is `alex/` or `matt/`.
2. `git switch main && git pull --ff-only`.
3. Glance at the board: `gh issue list --label status:in-progress` and `gh pr list` — know what
   the *other* human's team is touching so you don't collide.
4. If your human said **"first run"**, do the First-run checklist below.

### First-run checklist (once per machine)

- Confirm the 4 project agents load: `claude agents` lists `planner`, `builder`, `reviewer`,
  `deployer` from `.claude/agents/`.
- Toolchain: once issue #2 has landed, `rokit install` and confirm every tool in `rokit.toml`
  runs. Before #2 lands, just confirm `rokit` itself is installed.
- `gh auth status` has `repo` + `workflow` scopes.
- Upstream reference: `../openfront-upstream` exists and is checked out at the SHA pinned in
  `docs/openfront/README.md`. Clone/checkout it if not. It lives **outside** the repo, always.
- Read `docs/OPEN-QUESTIONS.md`. List every question marked **Blocking** that's still open and
  offer to resolve them with your human now.
- Report readiness in 5 lines or less, then suggest the next issue to pick up.

## The pipeline

```
human request ──► ORCHESTRATOR ──► planner ──(questions?)──► human ──► planner ──► PLAN approved by human
                                                                                         │
                     ┌───────────────────────────────────────────────────────────────────┘
                     ▼
                  builder ──(blocked?)──► human ──► builder ──► DONE
                     ▲                                            │
                     └──── CHANGES_REQUESTED ◄── reviewer ◄───────┘
                                                    │ APPROVED (+ docs committed, post-mortem)
                                                    ▼
          human says "deploy" ──► deployer ──► PR open, CI green ──► ask human "merge?" ──► merge ──► staging
          human says "promote" ──► deployer ──► production workflow (GitHub approval) ──► live
```

One job = one GitHub issue = one branch = one PR = one folder `docs/work/<issue>-<slug>/`.

### 1. Start a job
- Every job has an issue. If the human's request has none, draft one (title, labels, 2–4 line
  body), confirm, `gh issue create`.
- Assign it to the human, add `status:in-progress`.
- `git switch -c <prefix><issue>-<slug>` from fresh `main`.
- Create `docs/work/<issue>-<slug>/` by copying `docs/work/_template/`.

### 2. Plan — `planner`
- Delegate with: issue number + body, branch, work folder path, and the human's words verbatim.
- If planner returns `STATUS: QUESTIONS`: answer from repo docs **only** if the answer is
  explicitly written there (cite the file). Otherwise ask your human — batch the questions,
  include planner's options + recommendation. Then resume the **same** planner with the answers
  (SendMessage), don't spawn a new one.
- Loop until `STATUS: READY`. Show the human the **plain-speak summary** and acceptance criteria.
  Building starts only after the human explicitly approves.
- A question that's really a cross-cutting product/tech decision → also record it in
  `docs/OPEN-QUESTIONS.md` / an ADR so the other machine learns it too.

### 3. Build — `builder`
- Delegate with the work folder path. Builder follows `PLAN.md`.
- `STATUS: BLOCKED` → same question loop as planning, then resume the same builder.
- `STATUS: DONE` → go to review.

### 4. Review — `reviewer`
- `CHANGES_REQUESTED` → resume the builder with `REVIEW.md` findings. After **2** review cycles
  without approval, stop and bring the human in.
- `APPROVED` → relay the **plain-speak post-mortem** to the human (don't paraphrase it into
  jargon). The job is now waiting for a deploy command.

### 5. Deploy — `deployer` (only on the human's command)
- Trigger words: "deploy", "ship it", "open the PR". Nothing else starts the deployer.
- Deployer pushes, opens the PR, waits for CI. If it returns `BLOCKED` (CI red, conflicts), route
  code problems to builder → reviewer, then back to deployer.
- `READY_TO_MERGE` → ask the human: "All checks green on PR #N — merge?" Only an explicit yes
  counts. Then resume the same deployer to merge and watch the staging deploy.
- **Production** is separate: only on "promote to production" (or equivalent). Deployer starts
  the production workflow; a human still approves it in GitHub.

### Fast path
Docs-only or typo changes may skip planner if the human agrees. **Nothing skips reviewer.**

### Subagent mechanics
- Subagents can't talk to the humans directly — every question comes back to you. That's by design.
- Resume an agent with SendMessage to keep its context; spawn fresh only for a new job.
- Each agent ends its reply with a `STATUS:` line. Route on that line, not on vibes.

## Coordinating Alex ↔ Matt

- Pull `main` before starting anything. Rebase your branch on `main` before review if `main` moved.
- If the other human has an open issue/PR touching the same `area:` or files, tell your human
  before planning.
- Never push to `main`. Never touch the other human's branch. Never force-push a branch that
  isn't yours.
- Decisions live in the repo (`docs/decisions/`, `docs/OPEN-QUESTIONS.md`), never only in chat —
  the other machine can't see your chat.

## Source-of-truth order

1. `docs/decisions/` (ADRs) and `docs/PRODUCT.md`
2. `docs/knowledge/` — systems we've documented ourselves
3. `lib/OpenFrontCore/` — our Luau port of OpenFront's core, for anything already ported
4. OpenFront upstream at the pinned SHA, plus https://openfront.wiki — for anything not yet
   ported or documented here
5. Ask the human

Once a system is marked `ported` or `diverged` in the ledger, our library and docs win over upstream.

## Talking to Alex / Matt

- Plain speak first, detail on request. They may step away mid-job — say clearly what you're
  waiting on.
- Batch questions; give options with a recommendation.
- Don't narrate every agent hop. Report at the gates: plan ready, blocked, review result,
  PR status, merged, deployed.

## Hard rules (full list in CONTRIBUTING.md)

- We port OpenFront under AGPL-3.0 (ADR 0001). Ported code goes **only** in `lib/OpenFrontCore/`,
  with the attribution header, following `lib/OpenFrontCore/README.md`.
- Never use OpenFront's logos/brand (`proprietary/` upstream) or RoWorld's assets — references only.
- Desktop-first at 1920×1080, mouse + keyboard (ADR 0002). Don't spend effort on touch/gamepad.
- Server is authoritative. Never trust the client.
- Secrets live in GitHub environments, never in the repo or in chat.
