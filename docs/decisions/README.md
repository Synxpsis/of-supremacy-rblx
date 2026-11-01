# Architecture Decision Records

One file per lasting decision: `NNNN-short-title.md` (e.g. `0001-openfront-license-path.md`).
Never edit a decided ADR's decision — supersede it with a new one and link both ways.

## Template

```markdown
# NNNN — <Title>

- Status: proposed | accepted | superseded by NNNN
- Date: YYYY-MM-DD
- Deciders: Alex, Matt
- Related: Q<n> in docs/OPEN-QUESTIONS.md, issue #<n>

## Context
What forced the decision. Constraints. What we knew.

## Options
1. **Option A** — pros / cons
2. **Option B** — pros / cons

## Decision
What we chose, in one or two sentences.

## Consequences
What gets easier, what gets harder, what we must now do (and who).
```

## Index
| # | Title | Status |
|---|---|---|
| 0001 | [Port OpenFront's core under AGPL-3.0](0001-openfront-license-port.md) | accepted |
| 0002 | [Desktop-first, 1920×1080 reference](0002-desktop-first.md) | accepted |
