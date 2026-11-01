# Supremacy Live! — Product

> Status: v0 brief. Humans own this file. Agents propose changes via a docs PR; they don't edit
> it as a side effect of a feature job.

## One line
A 12–24 player, real-time, screens-only map supremacy game on Roblox: grab land, build an
economy, make and break alliances, and conquer everyone else — in one sitting.

## What it is
- **Screens-only.** No avatar, no 3D world. The whole game is a 2D map plus GUI panels.
- **Real-time PvP.** Everyone acts at once on a shared map; no turns.
- **Built on OpenFront's simulation.** Territory, troops, gold, structures, and diplomacy run on
  `OpenFrontCore`, our own Luau port of OpenFront's core — see `lib/OpenFrontCore/` and
  `docs/openfront/`. The game is AGPL-3.0 with OpenFront's attribution terms (ADR 0001).
- **Desktop-first.** Built for mouse + keyboard at 1920×1080 (ADR 0002).
- **References:** OpenFront (https://openfront.io, https://openfront.wiki) for mechanics;
  RoWorld (https://www.roblox.com/games/137151641329864/RoWorld) for how this genre feels on
  Roblox. We borrow no assets or branding from either.

## Pillars
1. **Quality over quantity.** Fewer features, each one polished, readable at a glance, and
   snappy with mouse and keyboard.
2. **Generous free-to-play.** Everything that affects winning is earnable by playing. Money buys
   looks, convenience that doesn't affect match outcomes, and private servers — never power.
3. **Progression that respects time.** Account levels, ranked tiers, and unlocks that reward
   regular play without punishing missed days.
4. **Live competition.** In-match standings update in real time; global and seasonal leaderboards
   are visible from the lobby.
5. **Fair and exploit-resistant.** The server decides everything. Ranked is protected from alts,
   smurfs, and disconnect abuse.

## Core match loop (OpenFront-derived defaults — confirm per system in spike #3)
1. **Spawn** — pick a starting location on the map.
2. **Expand** — send troops into neutral land, then into enemies. Territory grows tile by tile.
3. **Economy** — population/troops regenerate; gold accrues; balance troops-at-home vs. attacking.
4. **Build** — structures that boost economy, defense, or reach (OpenFront has cities, ports,
   defense posts, missile silos, SAM sites, and naval units — exact set decided per system).
5. **Diplomacy** — request/accept/break alliances; betrayal has consequences.
6. **Win** — control a target share of the map, or be the last one standing.

Target match length, player counts, and win threshold are tuned in `src/shared/Config/` and
recorded in `docs/knowledge/` once decided.

## Players & lobbies
- 12–24 humans per match. Bots/nations (OpenFront has AI opponents) fill empty slots so matches
  start fast.
- Modes to start: free-for-all. Teams later.
- Platform: **desktop** (Windows/macOS), mouse + keyboard, designed at 1920×1080 and usable from
  1366×768 to 2560×1440. Mobile, tablet, and console aren't supported at MVP.
- Desktop strengths we lean into: dense information panels, hover tooltips, hotkeys,
  scroll-zoom and drag-pan on the map, right-click context actions.

## Progression, ranked, leaderboards
- **Account progression:** XP from matches → levels → cosmetic unlocks (territory colors/patterns,
  flags, borders, name plates, effects).
- **Ranked:** separate queue, visible tiers backed by a hidden skill rating (system chosen in
  Q11), seasons with resets and cosmetic rewards.
- **Leaderboards:** live in-match standings; global all-time, seasonal ranked, and weekly boards.
- **Stats:** per-player match history and lifetime stats on a profile screen.

## Monetization guardrails
- Cosmetics, optional season pass, private servers. No paid power, no paid troop/gold boosts in
  ranked.
- Any paid random item must disclose odds per Roblox policy. Prefer direct purchases.
- Specifics in Q13.

## MVP (first playable)
One map · FFA · 12–24 players with bot fill · spawn, expand, troops/gold, 2–3 structures,
alliances, win condition · in-match leaderboard · basic profile with XP/level persisted ·
desktop UI on the design system with hotkeys · loading screen + Credits with OpenFront
attribution and our source link · staging deploy pipeline.

**Not in MVP:** ranked, seasons, monetization, teams, nukes/advanced units, multiple maps,
replays, spectate, mobile/console support.

## Non-goals
- A 3D world or avatar gameplay.
- Copying OpenFront's or RoWorld's art, names, or branding.
- Pay-to-win anything.

## Open product decisions
See `docs/OPEN-QUESTIONS.md` — notably Q11 (ranked rating), Q12 (game name), Q13
(monetization specifics), Q15 (how players get our source).
