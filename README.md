# Free Tycoon

> A roguelike party game about building software with the wrong people in the room.

**Free Tycoon** is a multiplayer roguelike that crosses the **dopamine loop of deck-building roguelikes** (Balatro, Slay the Spire) with the **couch-and-phone social chaos of Jackbox**. You and up to four others each play a role in a fictional software company — **DEV, CEO, Sales, HR, Product Management** — and over **5 rounds** you collectively (and contentiously) build a product. At the end, an LLM judge writes a verdict and scores what you shipped. Play solo against the CPU, or in co-op where the seats you don't fill are played by everyone else.

The hook: the first two rounds feel like a normal software project. Rounds 3–5 are **generated live by an LLM** seeded on the choices you already made — so every run drifts somewhere new, and the product you end up with is genuinely a surprise.

---

## The 30-second pitch

- **Genre:** Roguelike × party game (Jackbox topology: one shared screen, everyone joins on their phone with a room code).
- **Players:** 1 (solo vs CPU) to 5 (co-op).
- **Length:** One *run* = 5 rounds ≈ 10–15 minutes.
- **Core fantasy:** "We're a startup. We have no idea what we're doing. Let's ship it anyway."
- **Replay engine:** Rounds 1–2 are fixed/authored; rounds 3–5 are LLM-generated from your run's context. High variance → high replayability.
- **The payoff:** A fake-tech-press **product verdict** + a multi-axis **Software Score** that's different every time.

---

## How it reads (layered docs)

This repo is the single source of truth for the hackathon build. It's layered so you can read the top and stop, or read all the way down to schemas.

| Layer | Doc | Read this if you want… |
|-------|-----|------------------------|
| **1. Concept** | this README | the vision, the loop, the feel |
| **2. Game design** | [`docs/GAME_DESIGN.md`](docs/GAME_DESIGN.md) | roles, encounters, voting, scoring, progression — the rules |
| **3. Build spec** | [`docs/BUILD_SPEC.md`](docs/BUILD_SPEC.md) | data models, state machine, tech stack, MVP phasing |
| **4. LLM design** | [`docs/LLM_DESIGN.md`](docs/LLM_DESIGN.md) | generation prompts, output schemas, guardrails, fallbacks |
| **Research** | [`research/COMPETITIVE_ANALYSIS.md`](research/COMPETITIVE_ANALYSIS.md) | mechanics impulses from Balatro, StS, 9 Kings, Jackbox |

---

## The core loop (one run)

```
START RUN
  └─ Each player picks a ROLE (DEV / CEO / Sales / HR / PM)
      unfilled roles → CPU

  ROUND 1  ─┐
  ROUND 2  ─┤  authored content (fixed encounters/questions/boni)
  ROUND 3  ─┐
  ROUND 4  ─┤  LLM-generated, seeded by rounds 1–2 context
  ROUND 5  ─┘
        each round:
          1. CHOOSE   → player picks which other role to have an encounter with
          2. ENCOUNTER→ the two roles each pick from THEIR OWN stat-flavored options
          3. VOTE     → all other roles vote on the two candidate picks (majority wins)
          4. RESOLVE  → winning pick mutates the PRODUCT STATE
          5. BONI     → pick 1 of N suggested bonuses → carried into next round

  END RUN
  └─ SOFTWARE SCORE: multi-axis score + LLM-narrated product verdict
  └─ META-PROGRESSION: spend earned currency on unlocks for future runs (light, post-MVP)
```

Two progression layers, like every good roguelike:
- **Run progression** — boni stack across the 5 rounds (the Balatro "joker" loop).
- **Meta progression** — currency from completed runs buys persistent unlocks (the Slay the Spire "ascension/unlock" loop). *Intentionally de-scoped for the hackathon MVP.*

---

## Status

🏗️ **Hackathon project** (Benedict + Dennis). Design-complete, pre-implementation. See [`docs/BUILD_SPEC.md`](docs/BUILD_SPEC.md) for the phased build plan.

## License

TBD.
