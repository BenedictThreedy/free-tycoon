# Free Tycoon — Design Decisions

A lightweight log of design bets we've made deliberately — each with the **reasoning**, the **current choice**, and the **fallback** we'd switch to if the current one feels off in playtest. These are *decisions to revisit*, not locked rules.

---

## DR-001 — Bonus stacking: multiplicative + rising bar

**Status:** 🟡 Current bet (try first; fallback ready)
**Date:** 2026-05-30
**Affects:** [`GAME_DESIGN.md`](GAME_DESIGN.md) §4–5, [`BUILD_SPEC.md`](BUILD_SPEC.md) §2, §4

### Decision
Boni carry **flat** effects (additive axis nudges) **and multipliers** that multiply per-round axis gains. **Multipliers stack multiplicatively** (two ×1.5 → ×2.25). Rounds have a **rising score bar** (~×1.4/round) that the snowball is meant to outpace.

### Why
[Competitive research](../research/COMPETITIVE_ANALYSIS.md) found the Balatro **Chips × Mult** loop — multiplicative compounding against an exponentially-rising blind — is the single biggest "one more run" dopamine driver. Purely additive boni produce a flat, linear run with no escalation or tension. The multiplier snowball + rising bar is what makes a late-run bonus draft feel consequential ("watch the build explode").

### Fallback (if it feels off)
If multiplicative stacking proves **too swingy / hard to balance** in a hackathon timeframe (runaway snowballs, dead runs when the snowball doesn't start, or the math is opaque to players at the table):

→ **Revert to purely additive boni** (flat axis nudges only) with a **gentler, linear-ish bar**. Simpler to balance and reason about; loses some escalation but keeps the loop legible. This is the original as-conceived design and is a safe floor.

**Watch for in playtest:** Does the run feel like it escalates? Can a table *read* why a score jumped? Are runs decided by round 2's bonus, or stay live to round 5? If runs feel pre-decided or the math is illegible → take the fallback.

> Tension noted in research: roguelike *mastery* wants a legible system; if multiplicative math becomes table-unreadable, it erodes the very replayability it's meant to create. Legibility is the tie-breaker.
