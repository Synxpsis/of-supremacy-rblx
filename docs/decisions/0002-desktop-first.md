# 0002 — Desktop-first, 1920×1080 reference

- Status: accepted
- Date: 2026-10-31
- Deciders: Alex, Matt
- Related: Q14 in docs/OPEN-QUESTIONS.md

## Context
A screens-only supremacy game is dense: a large map plus many panels, numbers, and hotkeys.
Designing for every Roblox device at once would dilute the UI and slow everything down.

## Decision
Target **desktop, mouse + keyboard**. Design reference resolution is **1920×1080**. Mobile,
tablet, and console are unsupported at MVP (the game may still launch on them, but we don't
design or test for them).

## Consequences
- Layouts are designed at 1920×1080 and must stay usable from **1366×768 up to 2560×1440**
  (and ultrawide without stretching panels).
- Hover states, tooltips, right-click, scroll-to-zoom, drag-to-pan, and hotkeys are core interactions.
- Every action reachable by mouse; frequent actions also get a hotkey (rebindable later).
- No touch or gamepad requirements in reviews. Revisit with a new ADR if we expand platforms.
